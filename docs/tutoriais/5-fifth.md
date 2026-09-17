# Tutorial 5 — `fifth.cc`: rastreando a janela de congestionamento TCP

[← Anterior: fourth.cc](4-fourth.md) · [Índice dos tutoriais](README.md) · [Próximo: sixth.cc →](6-sixth.md)

`fifth.cc` (`examples/tutorial/fifth.cc`, junto com `tutorial-app.h`/`tutorial-app.cc`) aplica o mecanismo de trace visto em `fourth.cc` a um cenário de rede real: um fluxo TCP entre dois nós, rastreando a evolução da janela de congestionamento (`CongestionWindow`).

## O que ele introduz

- Um fluxo de dados **TCP real** (em vez do UDP echo dos exemplos anteriores).
- Uma **`Application` customizada** (`TutorialApp`), necessária para resolver um problema de ordem de eventos: o socket TCP normalmente só é criado quando a aplicação inicia — tarde demais para conectar um trace a ele via helpers comuns. A `TutorialApp` recebe um socket **já criado e com o trace já conectado** na fase de configuração, e só o usa depois, no início da simulação.
- Um **`RateErrorModel`**, que injeta perda de pacotes para gerar retransmissões e mudanças de janela mais interessantes de observar.
- `PacketSinkHelper`, um sumidouro (sink) de dados TCP.

## Topologia

```
        node 0                 node 1
  +----------------+    +----------------+
  |    ns-3 TCP    |    |    ns-3 TCP    |
  +----------------+    +----------------+
  |    10.1.1.1    |    |    10.1.1.2    |
  +----------------+    +----------------+
  | point-to-point |    | point-to-point |
  +----------------+    +----------------+
          |                     |
          +---------------------+
               5 Mbps, 2 ms
```

Dois nós, um único link ponto-a-ponto (5 Mbps, 2 ms — como no `first.cc`), mas endereçados com máscara `/30` (`255.255.255.252`) em vez de `/24`.

## Código

```cpp
#include "tutorial-app.h"

#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("FifthScriptExample");

static void
CwndChange(uint32_t oldCwnd, uint32_t newCwnd)
{
    NS_LOG_UNCOND(Simulator::Now().GetSeconds() << "\t" << newCwnd);
}

static void
RxDrop(Ptr<const Packet> p)
{
    NS_LOG_UNCOND("RxDrop at " << Simulator::Now().GetSeconds());
}

int
main(int argc, char* argv[])
{
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

    Config::SetDefault("ns3::TcpL4Protocol::SocketType", StringValue("ns3::TcpNewReno"));
    Config::SetDefault("ns3::TcpSocket::InitialCwnd", UintegerValue(1));
    Config::SetDefault("ns3::TcpL4Protocol::RecoveryType",
                       TypeIdValue(TypeId::LookupByName("ns3::TcpClassicRecovery")));

    NodeContainer nodes;
    nodes.Create(2);

    PointToPointHelper pointToPoint;
    pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

    NetDeviceContainer devices;
    devices = pointToPoint.Install(nodes);

    Ptr<RateErrorModel> em = CreateObject<RateErrorModel>();
    em->SetAttribute("ErrorRate", DoubleValue(0.00001));
    devices.Get(1)->SetAttribute("ReceiveErrorModel", PointerValue(em));

    InternetStackHelper stack;
    stack.SetIpv6StackInstall(false);
    stack.Install(nodes);

    Ipv4AddressHelper address;
    address.SetBase("10.1.1.0", "255.255.255.252");
    Ipv4InterfaceContainer interfaces = address.Assign(devices);

    uint16_t sinkPort = 8080;
    Address sinkAddress(InetSocketAddress(interfaces.GetAddress(1), sinkPort));
    PacketSinkHelper packetSinkHelper("ns3::TcpSocketFactory",
                                      InetSocketAddress(Ipv4Address::GetAny(), sinkPort));
    ApplicationContainer sinkApps = packetSinkHelper.Install(nodes.Get(1));
    sinkApps.Start(Seconds(0.));
    sinkApps.Stop(Seconds(20.));

    Ptr<Socket> ns3TcpSocket = Socket::CreateSocket(nodes.Get(0), TcpSocketFactory::GetTypeId());
    ns3TcpSocket->TraceConnectWithoutContext("CongestionWindow", MakeCallback(&CwndChange));

    Ptr<TutorialApp> app = CreateObject<TutorialApp>();
    app->Setup(ns3TcpSocket, sinkAddress, 1040, 1000, DataRate("1Mbps"));
    nodes.Get(0)->AddApplication(app);
    app->SetStartTime(Seconds(1.));
    app->SetStopTime(Seconds(20.));

    devices.Get(1)->TraceConnectWithoutContext("PhyRxDrop", MakeCallback(&RxDrop));

    Simulator::Stop(Seconds(20));
    Simulator::Run();
    Simulator::Destroy();

    return 0;
}
```

## Explicação bloco a bloco (o que muda em relação aos exemplos anteriores)

- **`Config::SetDefault(...)`** — define valores padrão de atributos globalmente, antes de qualquer objeto ser criado. Aqui força o tipo de socket para `TcpNewReno` e o algoritmo de recuperação para `TcpClassicRecovery` (usado apenas para demonstrar como parâmetros TCP podem ser configurados — as versões recentes do ns-3 usam Cubic/Prr por padrão).
- **`RateErrorModel` + `devices.Get(1)->SetAttribute("ReceiveErrorModel", PointerValue(em))`** — injeta perda de pacotes no dispositivo receptor (taxa de erro de 0.001%), o suficiente para provocar retransmissões e tornar a evolução da `CongestionWindow` mais interessante de observar.
- **`PacketSinkHelper("ns3::TcpSocketFactory", InetSocketAddress(Ipv4Address::GetAny(), sinkPort))`** — uma aplicação genérica de recepção de dados em massa, usando a fábrica de sockets TCP (padrão "object factory" via string de `TypeId`).
- **`Socket::CreateSocket(node, TcpSocketFactory::GetTypeId())`** — cria o socket TCP explicitamente, na fase de configuração (diferente dos exemplos anteriores, onde os helpers criam os sockets internamente).
- **`ns3TcpSocket->TraceConnectWithoutContext("CongestionWindow", MakeCallback(&CwndChange))`** — conecta o trace `CongestionWindow` do socket **antes** de a aplicação começar a rodar — exatamente o problema resolvido pela `TutorialApp`.
- **`TutorialApp : public Application`** — classe de aplicação customizada (definida em `tutorial-app.h`/`.cc`) que sobrescreve `StartApplication()`/`StopApplication()`. Recebe o socket já pronto via `Setup(socket, endereço, tamanhoPacote, nPacotes, taxaDeDados)` e, ao iniciar, faz `Bind()`, `Connect()` e agenda o envio dos pacotes.
- **`devices.Get(1)->TraceConnectWithoutContext("PhyRxDrop", MakeCallback(&RxDrop))`** — conecta a fonte de trace de descarte de pacotes na camada física do dispositivo receptor.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 run fifth
```

Para gerar um gráfico da janela de congestionamento ao longo do tempo:

```shell
./ns3 run fifth > cwnd.dat 2>&1
gnuplot
  set terminal png size 640,480
  set output "cwnd.png"
  plot "cwnd.dat" using 1:2 title 'Congestion Window' with linespoints
```

## Saída esperada

Saída não estruturada via `NS_LOG_UNCOND` (tempo em segundos + tamanho da nova janela, em bytes):

```
1.00419 536
1.0093  1072
...
RxDrop at 1.13696
```

Este exemplo não gera arquivos de trace estruturados — esse é exatamente o problema que o `sixth.cc` resolve.

---

[← Anterior: fourth.cc](4-fourth.md) · [Índice dos tutoriais](README.md) · [Próximo: sixth.cc →](6-sixth.md)

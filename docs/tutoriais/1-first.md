# Tutorial 1 — `first.cc`: o primeiro script ns-3

[← Índice dos tutoriais](README.md) · [Próximo: second.cc →](2-second.md)

Este é o primeiro exemplo do tutorial oficial do ns-3, localizado em `examples/tutorial/first.cc` (há também uma versão em Python equivalente, `first.py`, no mesmo diretório). Ele cria a simulação mais simples possível: dois nós ligados por um link ponto-a-ponto, trocando um único pacote UDP via uma aplicação de eco (echo).

## O que ele introduz

`first.cc` apresenta o padrão básico de qualquer script ns-3: criar nós, instalar dispositivos de rede e um canal entre eles, instalar a pilha de protocolos Internet, atribuir endereços IP, instalar aplicações e rodar o simulador. Todo o resto dos tutoriais constrói em cima deste padrão.

## Topologia

```
       10.1.1.0
n0 -------------- n1
   point-to-point
```

Dois nós (`n0`, `n1`) ligados por um único link ponto-a-ponto na sub-rede `10.1.1.0/24` (n0 = `10.1.1.1`, n1 = `10.1.1.2`), com taxa de 5 Mbps e atraso de propagação de 2 ms. O servidor de eco roda em n1 (porta 9); o cliente roda em n0 e envia 1 pacote de 1024 bytes.

## Código

```cpp
#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("FirstScriptExample");

int
main(int argc, char* argv[])
{
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

    Time::SetResolution(Time::NS);
    LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
    LogComponentEnable("UdpEchoServerApplication", LOG_LEVEL_INFO);

    NodeContainer nodes;
    nodes.Create(2);

    PointToPointHelper pointToPoint;
    pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

    NetDeviceContainer devices;
    devices = pointToPoint.Install(nodes);

    InternetStackHelper stack;
    stack.SetIpv6StackInstall(false);
    stack.Install(nodes);

    Ipv4AddressHelper address;
    address.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces = address.Assign(devices);

    UdpEchoServerHelper echoServer(9);
    ApplicationContainer serverApps = echoServer.Install(nodes.Get(1));
    serverApps.Start(Seconds(1));
    serverApps.Stop(Seconds(10));

    UdpEchoClientHelper echoClient(interfaces.GetAddress(1), 9);
    echoClient.SetAttribute("MaxPackets", UintegerValue(1));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(1)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(nodes.Get(0));
    clientApps.Start(Seconds(2));
    clientApps.Stop(Seconds(10));

    Simulator::Run();
    Simulator::Destroy();
    return 0;
}
```

## Explicação bloco a bloco

- **`CommandLine cmd(__FILE__); cmd.Parse(argc, argv);`** — habilita passar argumentos de linha de comando para o script (usado a partir do `second.cc` em diante).
- **`Time::SetResolution(Time::NS)`** — define a menor unidade de tempo representável pelo simulador (nanossegundo). Só pode ser chamado uma vez.
- **`LogComponentEnable(...)`** — liga o log em nível `INFO` dos componentes `UdpEchoClientApplication`/`UdpEchoServerApplication`, o que produz as linhas "sent"/"received" no console.
- **`NodeContainer nodes; nodes.Create(2);`** — helper de topologia: cria e gerencia um conjunto de objetos `Node`. Aqui cria os dois nós da simulação.
- **`PointToPointHelper`** — configura e conecta pares de `PointToPointNetDevice` + `PointToPointChannel`. `SetDeviceAttribute("DataRate", "5Mbps")` define a taxa da interface; `SetChannelAttribute("Delay", "2ms")` define o atraso de propagação do canal.
- **`pointToPoint.Install(nodes)`** — para os dois nós do container, cria um `PointToPointNetDevice` em cada um, cria o `PointToPointChannel` e conecta os dois dispositivos a ele.
- **`InternetStackHelper` / `stack.Install(nodes)`** — instala a pilha de protocolos Internet (TCP, UDP, IP etc.) em cada nó. `SetIpv6StackInstall(false)` desliga a pilha IPv6, que não é necessária neste exemplo (redes reais normalmente têm IPv4 e IPv6 juntos).
- **`Ipv4AddressHelper` / `SetBase("10.1.1.0", "255.255.255.0")` / `address.Assign(devices)`** — aloca endereços IP sequencialmente a partir de `.1` na sub-rede indicada e devolve um `Ipv4InterfaceContainer`, que associa cada IP ao seu dispositivo de rede.
- **`UdpEchoServerHelper echoServer(9)`** — configura uma aplicação de servidor de eco UDP escutando na porta 9; instalada no nó 1 (`nodes.Get(1)`), ativa entre os segundos 1 e 10.
- **`UdpEchoClientHelper echoClient(interfaces.GetAddress(1), 9)`** — configura o cliente de eco, apontando para o IP do servidor (`interfaces.GetAddress(1)` = `10.1.1.2`) e porta 9. `MaxPackets`, `Interval` e `PacketSize` controlam quantos pacotes enviar, o intervalo entre eles e o tamanho de cada um.
- **`ApplicationContainer::Start/Stop(Seconds(...))`** — agenda eventos no simulador para iniciar/parar cada aplicação.
- **`Simulator::Run()` / `Simulator::Destroy()`** — executa o laço de eventos discretos até não haver mais eventos agendados, depois libera todos os objetos.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 run first
```

Para editar o script livremente sem mexer no exemplo original, copie-o para a pasta `scratch/`:

```shell
cp examples/tutorial/first.cc scratch/myfirst.cc
./ns3 build
./ns3 run scratch/myfirst
```

## Saída esperada

```
At time +2s client sent 1024 bytes to 10.1.1.2 port 9
At time +2.00369s server received 1024 bytes from 10.1.1.1 port 49153
At time +2.00369s server sent 1024 bytes to 10.1.1.1 port 49153
At time +2.00737s client received 1024 bytes from 10.1.1.2 port 9
```

Este exemplo não gera arquivos de trace (`.pcap`/`.tr`) por si só — isso é adicionado manualmente em cima dele no capítulo "Tweaking" da documentação oficial do ns-3.

---

[← Índice dos tutoriais](README.md) · [Próximo: second.cc →](2-second.md)

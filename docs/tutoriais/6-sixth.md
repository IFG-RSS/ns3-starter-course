# Tutorial 6 — `sixth.cc`: gravando o trace em arquivos reais

[← Anterior: fifth.cc](5-fifth.md) · [Índice dos tutoriais](README.md) · [Próximo: seventh.cc →](7-seventh.md)

`sixth.cc` (`examples/tutorial/sixth.cc`) usa exatamente o mesmo cenário do `fifth.cc`, mas substitui os prints soltos no console (`std::cout`/`NS_LOG_UNCOND`) por arquivos de trace estruturados de verdade — um arquivo ASCII com a evolução da janela de congestionamento e um PCAP real com os pacotes descartados.

## O que ele introduz

- **`AsciiTraceHelper`**, para gerar um arquivo de saída ASCII a partir de um trace sink.
- **`PcapHelper`**, para gerar um arquivo PCAP real diretamente, sem passar por um `NetDeviceHelper`.
- **`MakeBoundCallback`**, que "amarra" um parâmetro extra (aqui, o stream/arquivo de destino) à frente da assinatura do callback.

A topologia é idêntica à do [`fifth.cc`](5-fifth.md): dois nós, link ponto-a-ponto de 5 Mbps/2 ms, endereçamento `10.1.1.0/30`.

## Código

```cpp
#include "tutorial-app.h"

#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("SixthScriptExample");

static void
CwndChange(Ptr<OutputStreamWrapper> stream, uint32_t oldCwnd, uint32_t newCwnd)
{
    NS_LOG_UNCOND(Simulator::Now().GetSeconds() << "\t" << newCwnd);
    *stream->GetStream() << Simulator::Now().GetSeconds() << "\t" << oldCwnd << "\t" << newCwnd
                         << std::endl;
}

static void
RxDrop(Ptr<PcapFileWrapper> file, Ptr<const Packet> p)
{
    NS_LOG_UNCOND("RxDrop at " << Simulator::Now().GetSeconds());
    file->Write(Simulator::Now(), p);
}

int
main(int argc, char* argv[])
{
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

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

    Ptr<TutorialApp> app = CreateObject<TutorialApp>();
    app->Setup(ns3TcpSocket, sinkAddress, 1040, 1000, DataRate("1Mbps"));
    nodes.Get(0)->AddApplication(app);
    app->SetStartTime(Seconds(1.));
    app->SetStopTime(Seconds(20.));

    AsciiTraceHelper asciiTraceHelper;
    Ptr<OutputStreamWrapper> stream = asciiTraceHelper.CreateFileStream("sixth.cwnd");
    ns3TcpSocket->TraceConnectWithoutContext("CongestionWindow",
                                             MakeBoundCallback(&CwndChange, stream));

    PcapHelper pcapHelper;
    Ptr<PcapFileWrapper> file =
        pcapHelper.CreateFile("sixth.pcap", std::ios::out, PcapHelper::DLT_PPP);
    devices.Get(1)->TraceConnectWithoutContext("PhyRxDrop", MakeBoundCallback(&RxDrop, file));

    Simulator::Stop(Seconds(20));
    Simulator::Run();
    Simulator::Destroy();

    return 0;
}
```

## Explicação bloco a bloco (o que muda em relação ao `fifth.cc`)

- **`AsciiTraceHelper` + `CreateFileStream("sixth.cwnd")`** — retorna um `Ptr<OutputStreamWrapper>`, que envolve um `std::ofstream`. Esse wrapper existe porque o construtor de cópia de `std::ostream` é privado, incompatível com o sistema de callbacks do ns-3 (que precisa de semântica de valor).
- **`MakeBoundCallback(&CwndChange, stream)`** — como `MakeCallback`, mas "amarra" um parâmetro extra (`stream`) à frente da lista de argumentos do callback; muda a assinatura exigida do sink para `(Ptr<OutputStreamWrapper>, uint32_t, uint32_t)`.
- **`PcapHelper` + `pcapHelper.CreateFile("sixth.pcap", std::ios::out, PcapHelper::DLT_PPP)`** — cria diretamente um arquivo em formato PCAP real. `DLT_PPP` é o link-type para ponto-a-ponto (em contraste com `DLT_EN10MB` para CSMA ou `DLT_IEEE802_11` para Wi-Fi).
- **`file->Write(Simulator::Now(), p)`** — uma linha dentro do trace sink já grava o pacote no arquivo PCAP.
- A documentação oficial do ns-3 destaca que `PcapFileWrapper` é um `Object` completo, integrado ao sistema de atributos/config, enquanto `OutputStreamWrapper` é apenas um `SimpleRefCount<>` leve, não um `Object` — ou seja, nem todo `Ptr<algumaCoisa>` do ns-3 aponta para um `Object`.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 run sixth
```

## Saída esperada

O console mostra a mesma sequência de eventos do `fifth.cc` (via `NS_LOG_UNCOND`), mas agora dois arquivos novos são gerados no diretório atual:

- **`sixth.cwnd`** — arquivo texto separado por tabulação (`tempo  janelaAntiga  janelaNova`), legível com `cat`:
  ```
  1.00419	0	5360
  1.1548	5360	1576
  1.15978	1576	3752
  1.42104	3752	1576
  1.42602	1576	2144
  ```
- **`sixth.pcap`** — arquivo PCAP real, contendo apenas os pacotes descartados (já que está conectado ao trace `PhyRxDrop`), legível com `tcpdump -r sixth.pcap -nn -tt`.

---

[← Anterior: fifth.cc](5-fifth.md) · [Índice dos tutoriais](README.md) · [Próximo: seventh.cc →](7-seventh.md)

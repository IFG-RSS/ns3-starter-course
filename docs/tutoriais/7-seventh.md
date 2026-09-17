# Tutorial 7 — `seventh.cc`: coleta de dados e IPv6 opcional

[← Anterior: sixth.cc](6-sixth.md) · [Índice dos tutoriais](README.md)

`seventh.cc` (`examples/tutorial/seventh.cc`) é o último capítulo do tutorial oficial do ns-3. Parte do `sixth.cc` e acrescenta dois recursos independentes: suporte opcional a **IPv6** e o **framework de coleta de dados** do ns-3 (`Probe`, `GnuplotHelper`, `FileHelper`), uma alternativa de mais alto nível aos trace sinks escritos à mão.

## O que ele introduz

- Um segundo caminho de código para usar **IPv6** em vez de IPv4, selecionável via `--useIpv6`.
- **Probes** (`ns3::Ipv4PacketProbe`/`ns3::Ipv6PacketProbe`): objetos que "escutam" uma fonte de trace já existente (aqui, o trace `Tx` do `Ipv4L3Protocol`/`Ipv6L3Protocol`) e expõem seus próprios traces derivados (`OutputBytes`).
- **`GnuplotHelper`**: gera um gráfico gnuplot (contagem de bytes de pacotes ao longo do tempo) com poucas linhas de código.
- **`FileHelper`**: grava a mesma informação em arquivo de texto formatado.
- **Caminhos de configuração com curinga** (`"/NodeList/*/$ns3::Ipv4L3Protocol/Tx"`), que casam com a fonte de trace em **todos** os nós de uma vez.

A topologia é idêntica à do [`fifth.cc`](5-fifth.md)/[`sixth.cc`](6-sixth.md): dois nós, link ponto-a-ponto de 5 Mbps/2 ms — endereçados em IPv4 `10.1.1.0/24` ou, com `--useIpv6=1`, em IPv6 `2001:0000:f00d:cafe::/64`.

## Código

```cpp
#include "tutorial-app.h"

#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/stats-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("SeventhScriptExample");

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
    bool useV6 = false;

    CommandLine cmd(__FILE__);
    cmd.AddValue("useIpv6", "Use Ipv6", useV6);
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
    if (useV6)
    {
        stack.SetIpv4StackInstall(false);
    }
    else
    {
        stack.SetIpv6StackInstall(false);
    }
    stack.Install(nodes);

    uint16_t sinkPort = 8080;
    Address sinkAddress;
    Address anyAddress;
    std::string probeType;
    std::string tracePath;
    if (!useV6)
    {
        Ipv4AddressHelper address;
        address.SetBase("10.1.1.0", "255.255.255.0");
        Ipv4InterfaceContainer interfaces = address.Assign(devices);
        sinkAddress = InetSocketAddress(interfaces.GetAddress(1), sinkPort);
        anyAddress = InetSocketAddress(Ipv4Address::GetAny(), sinkPort);
        probeType = "ns3::Ipv4PacketProbe";
        tracePath = "/NodeList/*/$ns3::Ipv4L3Protocol/Tx";
    }
    else
    {
        Ipv6AddressHelper address;
        address.SetBase("2001:0000:f00d:cafe::", Ipv6Prefix(64));
        Ipv6InterfaceContainer interfaces = address.Assign(devices);
        sinkAddress = Inet6SocketAddress(interfaces.GetAddress(1, 1), sinkPort);
        anyAddress = Inet6SocketAddress(Ipv6Address::GetAny(), sinkPort);
        probeType = "ns3::Ipv6PacketProbe";
        tracePath = "/NodeList/*/$ns3::Ipv6L3Protocol/Tx";
    }

    PacketSinkHelper packetSinkHelper("ns3::TcpSocketFactory", anyAddress);
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
    Ptr<OutputStreamWrapper> stream = asciiTraceHelper.CreateFileStream("seventh.cwnd");
    ns3TcpSocket->TraceConnectWithoutContext("CongestionWindow",
                                             MakeBoundCallback(&CwndChange, stream));

    PcapHelper pcapHelper;
    Ptr<PcapFileWrapper> file =
        pcapHelper.CreateFile("seventh.pcap", std::ios::out, PcapHelper::DLT_PPP);
    devices.Get(1)->TraceConnectWithoutContext("PhyRxDrop", MakeBoundCallback(&RxDrop, file));

    // Usa o GnuplotHelper para plotar a contagem de bytes de pacotes ao longo do tempo
    GnuplotHelper plotHelper;
    plotHelper.ConfigurePlot("seventh-packet-byte-count",
                             "Packet Byte Count vs. Time",
                             "Time (Seconds)",
                             "Packet Byte Count");
    plotHelper.PlotProbe(probeType,
                         tracePath,
                         "OutputBytes",
                         "Packet Byte Count",
                         GnuplotAggregator::KEY_BELOW);

    // Usa o FileHelper para escrever a contagem de bytes de pacotes em arquivo
    FileHelper fileHelper;
    fileHelper.ConfigureFile("seventh-packet-byte-count", FileAggregator::FORMATTED);
    fileHelper.Set2dFormat("Time (Seconds) = %.3e\tPacket Byte Count = %.0f");
    fileHelper.WriteProbe(probeType, tracePath, "OutputBytes");

    Simulator::Stop(Seconds(20));
    Simulator::Run();
    Simulator::Destroy();

    return 0;
}
```

## Explicação bloco a bloco (o que muda em relação ao `sixth.cc`)

- **`stack.SetIpv4StackInstall(false)` / `stack.SetIpv6StackInstall(false)`** — alterna qual pilha é instalada conforme a flag `useV6`.
- **`Ipv6AddressHelper` / `Ipv6InterfaceContainer` / `Inet6SocketAddress` / `Ipv6Prefix(64)`** — equivalentes IPv6 da API de endereçamento IPv4 já vista; endereço base `2001:0000:f00d:cafe::`.
- **`GnuplotHelper`** — helper voltado à produção de gráficos gnuplot com o mínimo de código possível. `ConfigurePlot(prefixoArquivo, título, rótuloX, rótuloY)` seguido de `PlotProbe(probeType, tracePath, nomeDoTraceDoProbe, rótuloDaSérie, GnuplotAggregator::KEY_BELOW)`.
- **Classes `Probe`** (`ns3::Ipv4PacketProbe`/`ns3::Ipv6PacketProbe`) — "extraem" os dados de um objeto `Packet` observado; expõem seu próprio trace source (`OutputBytes`). A fonte de trace original (`Tx` do `Ipv4L3Protocol`, do tipo `TracedCallback<Ptr<const Packet>, Ptr<Ipv4>, uint32_t>`) é casada com um tipo de Probe compatível — o ns-3 também tem `ApplicationPacketProbe`, `DoubleProbe`, `TimeProbe`, `UintegerNProbe`, `BooleanProbe` para outros tipos de `TracedValue`.
- **`tracePath = "/NodeList/*/$ns3::Ipv4L3Protocol/Tx"`** — caminho de configuração com curinga (`*`) que casa com a fonte de trace `Tx` em **todos** os nós, ilustrando caminhos de config com múltiplas correspondências.
- **`FileHelper`** — variante do `GnuplotHelper` que grava texto formatado em vez de um gráfico; `ConfigureFile(prefixo, FileAggregator::FORMATTED)`, `Set2dFormat(...)`, `WriteProbe(probeType, tracePath, "OutputBytes")`.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 run "seventh --help"
./ns3 run seventh
./ns3 run "seventh --useIpv6=1"
```

Saída de `--help`:
```
Program Arguments:
    --useIpv6:  Use Ipv6 [false]
```

## Saída esperada

O console mostra a mesma sequência de `CwndChange`/`RxDrop` do `sixth.cc`. Além de `seventh.cwnd` e `seventh.pcap` (herdados do `sixth.cc`), são gerados:

- `seventh-packet-byte-count-0.txt`, `seventh-packet-byte-count-1.txt` — um arquivo por nó casado pelo caminho com curinga, gerados pelo `FileHelper`.
- `seventh-packet-byte-count.dat`, `.plt`, `.sh` — gerados pelo `GnuplotHelper`. Rode `sh seventh-packet-byte-count.sh` para de fato invocar o gnuplot e produzir `seventh-packet-byte-count.png` (a imagem não é gerada automaticamente, para permitir ajustar o `.plt` à mão antes).

Para inspecionar o PCAP: `tcpdump -r seventh.pcap -nn -tt`.

---

[← Anterior: sixth.cc](6-sixth.md) · [Índice dos tutoriais](README.md)

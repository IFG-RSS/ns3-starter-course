# Tutorial 3 — `third.cc`: rede Wi-Fi e mobilidade

[← Anterior: second.cc](2-second.md) · [Índice dos tutoriais](README.md) · [Próximo: fourth.cc →](4-fourth.md)

`third.cc` (`examples/tutorial/third.cc`, com versão Python `third.py`) parte do `second.cc` e acrescenta uma rede sem fio Wi-Fi (802.11), com um ponto de acesso e estações móveis — sendo, portanto, o primeiro exemplo diretamente ligado ao tema de **redes móveis** da disciplina.

## O que ele introduz

- Dispositivos sem fio **802.11** (Wi-Fi), com um ponto de acesso (AP) e estações (STA).
- **Modelos de mobilidade**, incluindo movimento aleatório das estações.
- A necessidade de um **`Simulator::Stop()`** explícito: como os beacons do Wi-Fi geram eventos que se auto-perpetuam, a simulação nunca terminaria "naturalmente" sem esse comando.

## Topologia

```
  Wifi 10.1.3.0
                AP
 *    *    *    *
 |    |    |    |    10.1.1.0
n5   n6   n7   n0 -------------- n1   n2   n3   n4
                  point-to-point  |    |    |    |
                                  ================
                                    LAN 10.1.2.0
```

O backbone ponto-a-ponto + LAN CSMA do `second.cc` é mantido (n0↔n1 em `10.1.1.0/24`; LAN CSMA em `10.1.2.0/24` com n1..n4). Uma célula Wi-Fi é pendurada na ponta esquerda: n0 passa a ser o **Access Point** da rede `10.1.3.0/24`, e `nWifi` estações (3 por padrão: n5, n6, n7) se conectam a ele, movendo-se aleatoriamente. O servidor de eco continua no último nó CSMA; o cliente passa a rodar na última estação Wi-Fi.

## Código

```cpp
#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/mobility-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/ssid.h"
#include "ns3/yans-wifi-helper.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("ThirdScriptExample");

int
main(int argc, char* argv[])
{
    bool verbose = true;
    uint32_t nCsma = 3;
    uint32_t nWifi = 3;
    bool tracing = false;

    CommandLine cmd(__FILE__);
    cmd.AddValue("nCsma", "Number of \"extra\" CSMA nodes/devices", nCsma);
    cmd.AddValue("nWifi", "Number of wifi STA devices", nWifi);
    cmd.AddValue("verbose", "Tell echo applications to log if true", verbose);
    cmd.AddValue("tracing", "Enable pcap tracing", tracing);
    cmd.Parse(argc, argv);

    if (nWifi > 18)
    {
        std::cout << "nWifi should be 18 or less; otherwise grid layout exceeds the bounding box"
                  << std::endl;
        return 1;
    }

    if (verbose)
    {
        LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
        LogComponentEnable("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

    NodeContainer p2pNodes;
    p2pNodes.Create(2);

    PointToPointHelper pointToPoint;
    pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

    NetDeviceContainer p2pDevices;
    p2pDevices = pointToPoint.Install(p2pNodes);

    NodeContainer csmaNodes;
    csmaNodes.Add(p2pNodes.Get(1));
    csmaNodes.Create(nCsma);

    CsmaHelper csma;
    csma.SetChannelAttribute("DataRate", StringValue("100Mbps"));
    csma.SetChannelAttribute("Delay", TimeValue(NanoSeconds(6560)));

    NetDeviceContainer csmaDevices;
    csmaDevices = csma.Install(csmaNodes);

    NodeContainer wifiStaNodes;
    wifiStaNodes.Create(nWifi);
    NodeContainer wifiApNode = p2pNodes.Get(0);

    YansWifiChannelHelper channel = YansWifiChannelHelper::Default();
    YansWifiPhyHelper phy;
    phy.SetChannel(channel.Create());

    WifiMacHelper mac;
    Ssid ssid = Ssid("ns-3-ssid");

    WifiHelper wifi;

    NetDeviceContainer staDevices;
    mac.SetType("ns3::StaWifiMac", "Ssid", SsidValue(ssid), "ActiveProbing", BooleanValue(false));
    staDevices = wifi.Install(phy, mac, wifiStaNodes);

    NetDeviceContainer apDevices;
    mac.SetType("ns3::ApWifiMac", "Ssid", SsidValue(ssid));
    apDevices = wifi.Install(phy, mac, wifiApNode);

    MobilityHelper mobility;
    mobility.SetPositionAllocator("ns3::GridPositionAllocator",
                                  "MinX", DoubleValue(0.0),
                                  "MinY", DoubleValue(0.0),
                                  "DeltaX", DoubleValue(5.0),
                                  "DeltaY", DoubleValue(10.0),
                                  "GridWidth", UintegerValue(3),
                                  "LayoutType", StringValue("RowFirst"));

    mobility.SetMobilityModel("ns3::RandomWalk2dMobilityModel",
                              "Bounds", RectangleValue(Rectangle(-50, 50, -50, 50)));
    mobility.Install(wifiStaNodes);

    mobility.SetMobilityModel("ns3::ConstantPositionMobilityModel");
    mobility.Install(wifiApNode);

    InternetStackHelper stack;
    stack.SetIpv6StackInstall(false);
    stack.Install(csmaNodes);
    stack.Install(wifiApNode);
    stack.Install(wifiStaNodes);

    Ipv4AddressHelper address;
    address.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer p2pInterfaces;
    p2pInterfaces = address.Assign(p2pDevices);

    address.SetBase("10.1.2.0", "255.255.255.0");
    Ipv4InterfaceContainer csmaInterfaces;
    csmaInterfaces = address.Assign(csmaDevices);

    address.SetBase("10.1.3.0", "255.255.255.0");
    address.Assign(staDevices);
    address.Assign(apDevices);

    UdpEchoServerHelper echoServer(9);
    ApplicationContainer serverApps = echoServer.Install(csmaNodes.Get(nCsma));
    serverApps.Start(Seconds(1));
    serverApps.Stop(Seconds(10));

    UdpEchoClientHelper echoClient(csmaInterfaces.GetAddress(nCsma), 9);
    echoClient.SetAttribute("MaxPackets", UintegerValue(1));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(1)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(wifiStaNodes.Get(nWifi - 1));
    clientApps.Start(Seconds(2));
    clientApps.Stop(Seconds(10));

    Ipv4GlobalRoutingHelper::PopulateRoutingTables();
    Simulator::Stop(Seconds(10));

    if (tracing)
    {
        phy.SetPcapDataLinkType(WifiPhyHelper::DLT_IEEE802_11_RADIO);
        pointToPoint.EnablePcapAll("third");
        phy.EnablePcap("third", apDevices.Get(0));
        csma.EnablePcap("third", csmaDevices.Get(0), true);
    }

    Simulator::Run();
    Simulator::Destroy();
    return 0;
}
```

## Explicação bloco a bloco (o que muda em relação ao `second.cc`)

- **`YansWifiChannelHelper::Default()` + `YansWifiPhyHelper` + `phy.SetChannel(channel.Create())`** — configura o modelo de canal físico sem fio compartilhado ("Yet Another Network Simulator" PHY para Wi-Fi).
- **`WifiMacHelper` + `Ssid ssid("ns-3-ssid")`** — configura os parâmetros da camada MAC. `mac.SetType("ns3::StaWifiMac", "Ssid", SsidValue(ssid), "ActiveProbing", BooleanValue(false))` configura o MAC das estações (não-AP), que escutam beacons em vez de fazer probing ativo; `mac.SetType("ns3::ApWifiMac", "Ssid", SsidValue(ssid))` configura o MAC do ponto de acesso.
- **`WifiHelper` + `wifi.Install(phy, mac, nodeContainer)`** — por padrão configura o padrão 802.11ax (Wi-Fi 6) com um algoritmo de controle de taxa compatível, e cria os `WifiNetDevice`s.
- **`MobilityHelper`** — `SetPositionAllocator("ns3::GridPositionAllocator", ...)` posiciona os nós inicialmente em uma grade 2D; `SetMobilityModel("ns3::RandomWalk2dMobilityModel", "Bounds", Rectangle(...))` faz as estações "andarem" aleatoriamente dentro de uma caixa delimitadora; `SetMobilityModel("ns3::ConstantPositionMobilityModel")` mantém o AP parado.
- **`Simulator::Stop(Seconds(10))`** — como os beacons do Wi-Fi continuam gerando eventos indefinidamente, é preciso forçar o fim da simulação. Precisa ser chamado **antes** de `Simulator::Run()`.
- **`phy.SetPcapDataLinkType(WifiPhyHelper::DLT_IEEE802_11_RADIO)` + `phy.EnablePcap(...)`** — captura PCAP com cabeçalhos radiotap, específica para Wi-Fi.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
cp examples/tutorial/third.cc scratch/mythird.cc
./ns3 run "scratch/mythird --tracing=1"
```

Parâmetros disponíveis: `--nCsma`, `--nWifi` (máximo 18, por causa do layout em grade), `--verbose`, `--tracing`.

## Saída esperada

```
At time +2s client sent 1024 bytes to 10.1.2.4 port 9
At time +2.00626s server received 1024 bytes from 10.1.3.3 port 49153
At time +2.00626s server sent 1024 bytes to 10.1.3.3 port 49153
At time +2.0175s client received 1024 bytes from 10.1.2.4 port 9
```

### Arquivos gerados (com `--tracing=1`)

- `third-0-0.pcap` — nó 0 no link ponto-a-ponto.
- `third-0-1.pcap` — captura Wi-Fi (modo monitor) no ponto de acesso, com link-type `IEEE802_11_RADIO`.
- `third-1-0.pcap` — nó 1 no link ponto-a-ponto.
- `third-1-1.pcap` — captura promíscua na LAN CSMA.

---

[← Anterior: second.cc](2-second.md) · [Índice dos tutoriais](README.md) · [Próximo: fourth.cc →](4-fourth.md)

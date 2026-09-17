# Tutorial 2 — `second.cc`: adicionando uma LAN CSMA e roteamento

[← Anterior: first.cc](1-first.md) · [Índice dos tutoriais](README.md) · [Próximo: third.cc →](3-third.md)

`second.cc` (`examples/tutorial/second.cc`, com versão Python `second.py`) parte do `first.cc` e acrescenta uma rede CSMA (barramento tipo Ethernet), transformando a simulação em uma internetwork de verdade — com duas sub-redes que precisam de roteamento entre si.

## O que ele introduz

- Uma **rede CSMA** (barramento compartilhado, como uma LAN Ethernet clássica).
- Uma **internetwork** real: dois segmentos de rede diferentes, exigindo roteamento entre eles.
- **Roteamento global automático**, via `Ipv4GlobalRoutingHelper::PopulateRoutingTables()`.
- Captura **PCAP promíscua**, que "fareja" todo o tráfego de um segmento a partir de um único dispositivo.

## Topologia

```
      10.1.1.0
n0 -------------- n1   n2   n3   n4
   point-to-point  |    |    |    |
                    ================
                      LAN 10.1.2.0
```

O link ponto-a-ponto n0↔n1 (`10.1.1.0/24`, igual ao `first.cc`) permanece. O nó n1 também participa de uma LAN CSMA em `10.1.2.0/24`, junto com `nCsma` nós extras (3 por padrão: n2, n3, n4) — ou seja, n1 tem duas interfaces de rede e atua como roteador entre as duas sub-redes. O servidor de eco roda no último nó da LAN CSMA (n4); o cliente roda em n0.

## Código

```cpp
#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/ipv4-global-routing-helper.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("SecondScriptExample");

int
main(int argc, char* argv[])
{
    bool verbose = true;
    uint32_t nCsma = 3;

    CommandLine cmd(__FILE__);
    cmd.AddValue("nCsma", "Number of \"extra\" CSMA nodes/devices", nCsma);
    cmd.AddValue("verbose", "Tell echo applications to log if true", verbose);
    cmd.Parse(argc, argv);

    if (verbose)
    {
        LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
        LogComponentEnable("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

    nCsma = nCsma == 0 ? 1 : nCsma;

    NodeContainer p2pNodes;
    p2pNodes.Create(2);

    NodeContainer csmaNodes;
    csmaNodes.Add(p2pNodes.Get(1));
    csmaNodes.Create(nCsma);

    PointToPointHelper pointToPoint;
    pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

    NetDeviceContainer p2pDevices;
    p2pDevices = pointToPoint.Install(p2pNodes);

    CsmaHelper csma;
    csma.SetChannelAttribute("DataRate", StringValue("100Mbps"));
    csma.SetChannelAttribute("Delay", TimeValue(NanoSeconds(6560)));

    NetDeviceContainer csmaDevices;
    csmaDevices = csma.Install(csmaNodes);

    InternetStackHelper stack;
    stack.SetIpv6StackInstall(false);
    stack.Install(p2pNodes.Get(0));
    stack.Install(csmaNodes);

    Ipv4AddressHelper address;
    address.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer p2pInterfaces;
    p2pInterfaces = address.Assign(p2pDevices);

    address.SetBase("10.1.2.0", "255.255.255.0");
    Ipv4InterfaceContainer csmaInterfaces;
    csmaInterfaces = address.Assign(csmaDevices);

    UdpEchoServerHelper echoServer(9);
    ApplicationContainer serverApps = echoServer.Install(csmaNodes.Get(nCsma));
    serverApps.Start(Seconds(1));
    serverApps.Stop(Seconds(10));

    UdpEchoClientHelper echoClient(csmaInterfaces.GetAddress(nCsma), 9);
    echoClient.SetAttribute("MaxPackets", UintegerValue(1));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(1)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(p2pNodes.Get(0));
    clientApps.Start(Seconds(2));
    clientApps.Stop(Seconds(10));

    Ipv4GlobalRoutingHelper::PopulateRoutingTables();

    pointToPoint.EnablePcapAll("second");
    csma.EnablePcap("second", csmaDevices.Get(1), true);

    Simulator::Run();
    Simulator::Destroy();
    return 0;
}
```

## Explicação bloco a bloco (o que muda em relação ao `first.cc`)

- **`cmd.AddValue("nCsma", ..., nCsma)`** — expõe `nCsma` como parâmetro de linha de comando (`--nCsma=N`), controlando quantos nós "extras" a LAN CSMA terá.
- **`csmaNodes.Add(p2pNodes.Get(1)); csmaNodes.Create(nCsma);`** — reaproveita o nó n1 (que já existe no link ponto-a-ponto) e o adiciona também ao container da LAN CSMA. É esse compartilhamento que faz n1 ter duas interfaces e funcionar como roteador entre as duas redes.
- **`CsmaHelper`** — funciona como o `PointToPointHelper`, mas cria e conecta dispositivos/canais CSMA. A taxa de dados é um atributo do **canal** (não do dispositivo), porque uma rede CSMA real não permite misturar dispositivos com taxas diferentes no mesmo barramento.
- **`stack.Install(p2pNodes.Get(0)); stack.Install(csmaNodes);`** — instala a pilha Internet em n0 e em todos os nós CSMA (que já incluem n1); evita instalar duas vezes em n1.
- **Dois `Ipv4AddressHelper::SetBase`** — uma sub-rede para cada segmento: `10.1.1.0/24` para o link ponto-a-ponto e `10.1.2.0/24` para a LAN CSMA.
- **`Ipv4GlobalRoutingHelper::PopulateRoutingTables()`** — faz cada nó se comportar como se fosse um roteador OSPF que troca rotas instantânea e "magicamente" com todos os outros, preenchendo as tabelas de roteamento automaticamente. Chamado uma única vez, depois que todos os endereços já foram atribuídos.
- **`pointToPoint.EnablePcapAll("second")`** — habilita captura PCAP em todos os dispositivos ponto-a-ponto.
- **`csma.EnablePcap("second", csmaDevices.Get(1), true)`** — habilita PCAP em um dispositivo CSMA específico; o último argumento (`true`) ativa o **modo promíscuo**: aquele dispositivo passa a "farejar" todos os pacotes do barramento e gravá-los em um único arquivo pcap — é assim que ferramentas como o `tcpdump` funcionam em uma rede real.

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
export NS_LOG=""
./ns3 run second
```

> O nome `second` já existe como um executável de teste de regressão do ns-3; ao copiar para `scratch/`, renomeie o arquivo para evitar conflito:

```shell
cp examples/tutorial/second.cc scratch/mysecond.cc
./ns3 build
./ns3 run scratch/mysecond
```

Parâmetros disponíveis:

```shell
./ns3 run "scratch/mysecond --nCsma=4"
```

## Saída esperada

```
At time +2s client sent 1024 bytes to 10.1.2.4 port 9
At time +2.0108s server received 1024 bytes from 10.1.1.1 port 49153
At time +2.0108s server sent 1024 bytes to 10.1.1.1 port 49153
At time +2.01861s client received 1024 bytes from 10.1.2.4 port 9
```

### Arquivos gerados

Convenção de nomes: `<prefixo>-<idDoNó>-<idDoDispositivo>.pcap`.

- `second-0-0.pcap` — nó 0 (ponto-a-ponto).
- `second-1-0.pcap` — nó 1 (o nó com as duas interfaces).
- `second-2-0.pcap` — captura promíscua no primeiro nó "extra" da LAN CSMA.

Para inspecionar: `tcpdump -nn -tt -r second-1-0.pcap`.

---

[← Anterior: first.cc](1-first.md) · [Índice dos tutoriais](README.md) · [Próximo: third.cc →](3-third.md)

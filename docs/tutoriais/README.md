# Tutoriais ns-3: first → seventh

Explicações detalhadas, página a página, da progressão clássica de exemplos do tutorial oficial do ns-3 (`examples/tutorial/` na árvore `ns-3.48`). Cada página cobre o que o exemplo introduz de novo, o código comentado, a topologia, como compilar/rodar e a saída esperada.

Pré-requisito: siga primeiro o [tutorial de instalação](../instalacao.md).

| # | Tutorial | O que introduz |
|---|---|---|
| 1 | [first.cc](1-first.md) | Simulação mínima: 2 nós, 1 link ponto-a-ponto, eco UDP |
| 2 | [second.cc](2-second.md) | + LAN CSMA, internetwork com 2 sub-redes, roteamento global, PCAP promíscuo |
| 3 | [third.cc](3-third.md) | + Rede Wi-Fi (802.11) com AP e estações móveis |
| 4 | [fourth.cc](4-fourth.md) | Mecanismo de trace source/trace sink do ns-3, sem rede |
| 5 | [fifth.cc](5-fifth.md) | Fluxo TCP real e rastreamento da janela de congestionamento |
| 6 | [sixth.cc](6-sixth.md) | O mesmo cenário do fifth, gravando trace em arquivos reais (ASCII/PCAP) |
| 7 | [seventh.cc](7-seventh.md) | + IPv6 opcional e framework de coleta de dados (Probes, gnuplot) |

Os exemplos ficam em `/home/rogerio/git/ns-allinone-3.48/ns-3.48/examples/tutorial/`.

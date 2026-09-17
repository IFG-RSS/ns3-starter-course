# Disciplina: Redes Móveis

Material da disciplina **Redes Móveis**, do curso de Bacharelado em Engenharia de Software do Instituto Federal de Goiás — Câmpus Inhumas.

## Dados gerais

| | |
|---|---|
| Curso | 03018 — Bacharelado em Engenharia de Software (Câmpus Inhumas) |
| Disciplina | DPAAIN.0108 — Redes Móveis |
| Carga horária | 54h / 72 aulas |
| Período letivo | 2026/2 |
| Professor | Rogério Sousa e Silva |

## Objetivo geral

Apresentar arquiteturas, serviços e protocolos de redes móveis sem fio, com foco em seu funcionamento, desempenho e aplicações, discutindo aspectos técnicos e práticos dessas tecnologias e analisando suas principais limitações e desafios.

## Objetivos específicos

- Compreender as características dos enlaces e redes sem fio.
- Explorar as restrições físicas e tecnológicas.
- Explicar os princípios da propagação via rádio.
- Analisar os protocolos de acesso ao meio e redes sem fio.
- Estudar as arquiteturas e protocolos de Wireless LAN/WAN, Packet Radio Networks e Mobile-IP.
- Estudar a mobilidade, segurança e o desempenho das redes móveis.
- Aplicar conhecimentos na implementação e simulação de redes móveis.

## Justificativa

A disciplina de Redes Móveis justifica-se pela centralidade dessas tecnologias na sociedade contemporânea, marcada pela mobilidade ubíqua e pela integração de sistemas distribuídos. O estudo de arquiteturas, protocolos e restrições físicas de redes sem fio possibilita compreender os fundamentos que sustentam aplicações modernas como 5G, IoT e computação em nuvem móvel. Além disso, a abordagem prática com ferramentas de simulação promove o desenvolvimento de competências analíticas e de projeto, essenciais à formação do engenheiro de software.

## Ementa / conteúdo programático

- Introdução às Redes Móveis e Sem Fio: definições, histórico, evolução e aplicações.
- Características de Enlaces e Redes Sem Fio: parâmetros de desempenho, desafios e oportunidades.
- Restrições Físicas e Tecnológicas: limitações de potência, interferências, variações de canal.
- Propagação via Rádio.
- Protocolos de Acesso ao Meio: CSMA/CA, CDMA, TDMA, FDMA, protocolos de controle de acesso ao meio.
- Packet Radio Networks: conceitos e tecnologias.
- Wireless LAN/WAN: padrões IEEE 802.11 (Wi-Fi), IEEE 802.16 (WiMAX), arquiteturas e protocolos.
- Mobile-IP: mobilidade e gerenciamento de IP em redes móveis.
- Protocolos de Redes Sem Fio: Bluetooth, ZigBee, NFC, LTE, 5G.
- Mobilidade de Sessão e Handoff.
- Segurança em Redes Móveis: autenticação, criptografia, proteção de dados.

## Metodologia

- **Aulas teóricas**: conceitos fundamentais de redes móveis.
- **Aulas práticas**: simulações e experimentos com ferramentas como ns-3 e Wireshark.
- **Estudos de caso**: análise de cenários reais e discussões (ex.: redes 5G, redes IoT móveis).
- **Seminários**: apresentação de protocolos e tecnologias emergentes por grupos de alunos.
- **Trabalhos em grupo**: desenvolvimento de projetos que abordem os desafios de implementar redes móveis.
- **Projeto final**: desenvolvimento e simulação de uma rede móvel no ns-3.

## Critérios avaliativos

Conforme o Plano de Ensino oficial da disciplina:

| Critério | Peso |
|---|---|
| Provas escritas (intermediária e final) | 40% |
| Trabalho em grupo (projeto prático de rede móvel, com simulação e análise de desempenho) | 30% |
| Seminários (estudos de caso sobre tecnologias emergentes) | 20% |
| Participação e atividades em sala | 10% |

## Cronograma de aulas

| Semana | Tópico | Atividade |
|---|---|---|
| 1 | Apresentação da disciplina e Introdução às Redes Móveis | Aula expositiva |
| 2 | Introdução às Redes Móveis | Aula expositiva |
| 3 | Características de Enlaces e Redes Sem Fio | Aula expositiva + atividade prática |
| 4 | Características de Enlaces e Redes Sem Fio | Aula expositiva + atividade prática |
| 5 | Restrições Físicas e Tecnológicas | Estudo de caso: redes Wi-Fi em ambientes externos |
| 6 | Propagação via Rádio | Simulação de propagação |
| 7 | Protocolos de Acesso ao Meio | Estudo de protocolos |
| 8 | Packet Radio Networks | Aula + atividade prática |
| 9 | Wireless LAN/WAN | Simulação de rede Wi-Fi |
| 10 | Prova 1 | Prova |
| 11 | Mobile-IP e Mobilidade | Aula expositiva + atividade prática |
| 12 | Protocolos de Redes Sem Fio | Apresentação de seminários |
| 13 | Mobilidade de Sessão e Handoff | Aula prática de simulação |
| 14 | Segurança em Redes Móveis | Atividade prática com foco em segurança |
| 15 | Prova 2 | Prova |
| 16 | Avaliação de Desempenho de Redes Móveis | Atividade prática com foco em desempenho |
| 17 | Projeto Final: Desenvolvimento de uma Rede Móvel | Início do projeto prático |
| 18 | Apresentação dos Projetos | Apresentação final |

## Ferramentas

- **ns-3** — simulador de redes usado nas atividades práticas e no projeto final (veja o [tutorial de instalação](instalacao.md) e os [tutoriais ns-3](tutoriais/)).
- **Wireshark** — análise de tráfego capturado nas simulações.
- **Iperf** — medição de desempenho de rede.

## Bibliografia

### Básica

- COULOURIS, George F. *Sistemas distribuídos: conceitos e projeto*. 4. ed. Porto Alegre: Bookman. ISBN 9788560031498.
- SCHILLER, Jochen. *Mobile communications*. 2. ed. Londres; Nova Iorque: Addison-Wesley, 2003. ISBN 9780321123817.
- TELLES, A.; KOLBE JÚNIOR, A. *Smart IoT: a revolução da internet das coisas para negócios inovadores*. Curitiba: Intersaberes, 2022.

### Complementar

- STALLINGS, W. *Wireless Communications and Networks*. 2. ed. Prentice Hall, 2005.
- AGRAWAL, D.; ZENG, Q. *Introduction to Wireless and Mobile Systems*. 4. ed. Cengage Learning, 2015.
- GOLDMAN, J. R. *Wireless Communications: Principles and Practice*. 2. ed. Prentice Hall, 2002.

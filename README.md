# ns3-starter-course

Material de apoio da disciplina **Redes Móveis** — Bacharelado em Engenharia de Software, Instituto Federal de Goiás (Câmpus Inhumas). Professor: Rogério Sousa e Silva.

## Sobre este repositório

Este repositório reúne o passo a passo de instalação do simulador de redes **ns-3** e explicações detalhadas dos exemplos clássicos do tutorial oficial do ns-3 (`first.cc` a `seventh.cc`), usados nas aulas práticas e no projeto final da disciplina.

## Sumário

- [Disciplina](#disciplina)
- [Instalação](#instalação)
- [Tutoriais ns-3](#tutoriais-ns-3)

## Disciplina

Ementa, objetivos, metodologia, critérios avaliativos, cronograma de aulas e bibliografia da disciplina Redes Móveis: veja [`docs/disciplina.md`](docs/disciplina.md).

## Instalação

Para instalar e validar o ns-3 (versão 3.48, distribuição `ns-3-allinone`), siga o [tutorial de instalação](docs/instalacao.md).

## Tutoriais ns-3

Explicações página a página dos sete exemplos clássicos do tutorial oficial do ns-3, do mais simples ao mais completo:

1. [first.cc](docs/tutoriais/1-first.md) — simulação mínima: 2 nós, 1 link ponto-a-ponto, eco UDP.
2. [second.cc](docs/tutoriais/2-second.md) — LAN CSMA, internetwork com 2 sub-redes, roteamento global.
3. [third.cc](docs/tutoriais/3-third.md) — rede Wi-Fi (802.11) com ponto de acesso e estações móveis.
4. [fourth.cc](docs/tutoriais/4-fourth.md) — mecanismo de trace source/trace sink do ns-3.
5. [fifth.cc](docs/tutoriais/5-fifth.md) — fluxo TCP real e rastreamento da janela de congestionamento.
6. [sixth.cc](docs/tutoriais/6-sixth.md) — o mesmo cenário do fifth, gravando trace em arquivos reais.
7. [seventh.cc](docs/tutoriais/7-seventh.md) — IPv6 opcional e framework de coleta de dados (Probes, gnuplot).

Veja o índice completo em [`docs/tutoriais/`](docs/tutoriais/README.md).

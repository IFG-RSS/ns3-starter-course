# Instalação do ns-3 (ns-3-allinone-3.48)

Este guia mostra como instalar e validar o **ns-3 versão 3.48**, distribuído no pacote `ns-3-allinone`, em Linux (Ubuntu/Debian).

> **Atenção:** a partir da versão 3.45, o pacote `ns-3-allinone` deixou de incluir o script `build.py`, o animador **NetAnim** e a ferramenta **bake**. Hoje o "allinone" significa apenas "ns-3 com um conjunto de módulos contrib (App Store) já populados em `contrib/`". Se você encontrar tutoriais antigos na internet mencionando `build.py` ou `waf`, ignore-os: nesta versão o build é feito inteiramente pelo script `ns3` (CMake), descrito abaixo.

## 1. Pré-requisitos

| Ferramenta | Versão mínima |
|---|---|
| `g++` | 11.1 ou superior (alternativamente `clang++` ≥ 17) |
| `python3` | 3.10 ou superior |
| `cmake` | 3.25 ou superior |
| `ninja` (ou `make`) | qualquer versão recente |
| `git` | qualquer versão recente |

## 2. Instalando as dependências (Ubuntu/Debian)

```shell
sudo apt update
sudo apt install g++ python3 cmake ninja-build git
```

Se quiser conferir as versões já instaladas na sua máquina antes de prosseguir:

```shell
g++ --version
python3 --version
cmake --version
ninja --version
```

## 3. Código-fonte

Este tutorial assume que a distribuição `ns-3-allinone-3.48` já está disponível localmente em:

```
/home/rogerio/git/ns-allinone-3.48/ns-3.48
```

Se você estiver reproduzindo este tutorial em outra máquina, clone o ns-3 (versão 3.48) a partir do repositório oficial antes de continuar.

## 4. Configurando e compilando

Entre na pasta do ns-3.48 e execute os comandos de configuração e build:

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 configure --enable-examples --enable-tests
./ns3 build
```

- `--enable-examples` e `--enable-tests` habilitam os exemplos e os testes usados nas próximas etapas.
- O resultado da compilação fica em `build/`.

A compilação completa pode demorar bastante na primeira vez (vários minutos), dependendo da máquina.

## 5. Verificando a instalação

Rode o exemplo `simple-global-routing`, indicado no `README.md` oficial do ns-3 como o teste de sanidade padrão:

```shell
./ns3 run simple-global-routing
```

Se tudo estiver correto, esse programa gera um arquivo de trace `simple-global-routing.tr` e arquivos binários `.pcap` (`simple-global-routing-xx-xx.pcap`). O código-fonte desse exemplo está em `examples/routing/`.

Também é possível rodar um exemplo ainda mais simples, só para confirmar que o build funciona:

```shell
./ns3 run hello-simulator
```

## 6. Rodando a suíte de testes

Para validar a instalação de forma mais completa, rode a suíte de testes do ns-3:

```shell
./test.py
```

Esse comando executa várias centenas de testes unitários. Se todos passarem, o build está funcionando corretamente.

## 7. Próximos passos

- Os binários compilados ficam em `build/`.
- Os exemplos didáticos oficiais do ns-3 (usados no restante do curso) estão em `examples/tutorial/`: `first.cc`, `second.cc`, `third.cc`, `fourth.cc`, `fifth.cc`, `sixth.cc` e `seventh.cc` (com versões em Python `first.py`, `second.py` e `third.py`).
- Para rodar qualquer exemplo: `./ns3 run <nome-do-exemplo>`.

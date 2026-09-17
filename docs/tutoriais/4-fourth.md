# Tutorial 4 — `fourth.cc`: o mecanismo de tracing do ns-3

[← Anterior: third.cc](3-third.md) · [Índice dos tutoriais](README.md) · [Próximo: fifth.cc →](5-fifth.md)

`fourth.cc` (`examples/tutorial/fourth.cc`) é uma ruptura proposital em relação aos três exemplos anteriores: não cria nenhum nó, rede ou evento de simulação. É o exemplo mais simples possível do mecanismo de **trace sources/trace sinks** do ns-3, isolado de qualquer topologia, para ensinar o conceito antes de aplicá-lo a um fluxo TCP real no `fifth.cc`.

## O que ele introduz

- O sistema de metatipos `Object`/`TypeId`, base do modelo de atributos do ns-3.
- **`TracedValue<T>`**: uma variável que, ao ser alterada, dispara automaticamente callbacks com o valor antigo e o novo.
- **`TraceConnectWithoutContext`**: como conectar uma função de callback ("trace sink") a uma fonte de trace ("trace source").

## Código

```cpp
#include "ns3/object.h"
#include "ns3/simulator.h"
#include "ns3/trace-source-accessor.h"
#include "ns3/traced-value.h"
#include "ns3/uinteger.h"

#include <iostream>

using namespace ns3;

class MyObject : public Object
{
  public:
    static TypeId GetTypeId()
    {
        static TypeId tid = TypeId("MyObject")
                                .SetParent<Object>()
                                .SetGroupName("Tutorial")
                                .AddConstructor<MyObject>()
                                .AddTraceSource("MyInteger",
                                                "An integer value to trace.",
                                                MakeTraceSourceAccessor(&MyObject::m_myInt),
                                                "ns3::TracedValueCallback::Int32");
        return tid;
    }

    MyObject()
    {
    }

    TracedValue<int32_t> m_myInt;
};

void
IntTrace(int32_t oldValue, int32_t newValue)
{
    std::cout << "Traced " << oldValue << " to " << newValue << std::endl;
}

int
main(int argc, char* argv[])
{
    Ptr<MyObject> myObject = CreateObject<MyObject>();
    myObject->TraceConnectWithoutContext("MyInteger", MakeCallback(&IntTrace));

    myObject->m_myInt = 1234;

    return 0;
}
```

## Explicação bloco a bloco

- **`class MyObject : public Object` + `GetTypeId()`** — todo objeto que participa do sistema de atributos/trace do ns-3 herda de `Object` e registra um `TypeId` estático. `.AddConstructor<MyObject>()` registra um construtor padrão no `TypeId`.
- **`TracedValue<int32_t> m_myInt;`** — fornece a infraestrutura que dispara o processo de callback: toda vez que o valor subjacente é alterado, o mecanismo `TracedValue` chama os callbacks conectados com o valor antigo e o novo.
- **`.AddTraceSource("MyInteger", texto, MakeTraceSourceAccessor(&MyObject::m_myInt), "ns3::TracedValueCallback::Int32")`** — registra a fonte de trace no sistema de configuração/atributos do ns-3. O último argumento (string) nomeia o typedef usado para documentar/impor a assinatura esperada do callback.
- **`myObject->TraceConnectWithoutContext("MyInteger", MakeCallback(&IntTrace))`** — forma a conexão entre a fonte de trace e o "sink" (função de callback). Mais adiante, no `fifth.cc`/`sixth.cc`, o mesmo padrão é usado em objetos reais da simulação (sockets, dispositivos de rede); existe também `Config::Connect`/`Config::ConnectWithoutContext`, que localizam fontes de trace por um caminho em string, sem precisar de um ponteiro direto ao objeto.
- **`myObject->m_myInt = 1234;`** — a atribuição via `operator=` de um `TracedValue` dispara todos os callbacks conectados, passando (valorAntigo, valorNovo).

## Como compilar e executar

```shell
cd /home/rogerio/git/ns-allinone-3.48/ns-3.48
./ns3 run fourth
```

## Saída esperada

```
Traced 0 to 1234
```

Não há arquivos de trace nem rede envolvidos — é uma demonstração pura via `std::cout`, cujo único propósito é fixar o conceito de trace source/trace sink antes de usá-lo em um cenário de rede real.

---

[← Anterior: third.cc](3-third.md) · [Índice dos tutoriais](README.md) · [Próximo: fifth.cc →](5-fifth.md)

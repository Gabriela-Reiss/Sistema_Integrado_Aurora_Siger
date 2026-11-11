# Sistema Integrado de Monitoramento e Tomada de Decisão da Colônia — Missão Aurora Siger

## Sobre o projeto

Com a necessidade de manter a colônia funcionando de maneira estável e eficiente, o sistema utiliza dados simulados de sensores para acompanhar as 
condições energéticas, ambientais e operacionais. A partir dessas informações, realiza cálculos, identifica possíveis riscos, analisa padrões históricos 
e apresenta recomendações para auxiliar nas decisões operacionais.

A proposta é demonstrar como a integração entre dados, regras lógicas e modelos matemáticos pode contribuir para a evolução de um sistema que não apenas observa 
as condições atuais, mas também utiliza informações históricas para apoiar decisões futuras.

----------------------

## Funcionamento geral do sistema

O sistema recebe um conjunto de leituras simuladas, representando informações coletadas pelos sensores distribuídos pela colônia.

Cada leitura contém informações referentes a um determinado horário e é processada por diferentes funções, responsáveis por analisar os dados e produzir resultados específicos.

O funcionamento segue o fluxo:

- Entrada de dados: recebimento das informações energéticas, ambientais e operacionais.
- Organização: armazenamento dos dados em estruturas de listas e dicionários.
- Processamento: execução dos cálculos e análises de cada subsistema.
- Previsão: utilização da regressão linear para estimar a geração eólica.
- Tomada de decisão: aplicação de regras condicionais para identificar riscos e gerar recomendações.
- Saída: apresentação de relatórios individuais, indicadores gerais e visualizações.

---

##  Organização e estrutura dos dados

A base de dados utilizada é fictícia e representa leituras realizadas de hora em hora durante um período de 24 horas.

As informações são organizadas em uma lista de dicionários, na qual cada elemento representa uma leitura. 
Dentro de cada leitura, os dados são agrupados em três categorias:

| Estrutura | Informações armazenadas |
|---|---|
| `energia` | Geração solar, geração eólica, consumo e percentual da bateria. |
| `ambiente` | Temperatura interna, temperatura externa e velocidade do vento. |
| `sistemas` | Estado operacional dos equipamentos da colônia. |

Exemplo simples da estrutura dos dados:

```python
{
    "horario": "12:00",

    "energia": {
        "solar": 46,
        "eolica": 30,
        "consumo": 58,
        "bateria": 78
    },

    "ambiente": {
        "temperatura_interna": 23,
        "temperatura_externa": -28,
        "vento": 12
    },

    "sistemas": {
        "suporte_vida": True,
        "agua": True,
        "controle_termico": True,
        "comunicacao": True,
        "laboratorio": True,
        "iluminacao": True
    }
}
```

As leituras energéticas representam valores acumulados em cada intervalo de uma hora, expressos em kWh. 
A velocidade do vento é expressa em m/s, as temperaturas em °C e o nível da bateria em percentual.

Essa organização permite acessar informações específicas de maneira estruturada, facilitando a análise individual e integrada dos subsistemas.

---

## Hierarquia e prioridade dos sistemas

Além das informações dos sensores, o sistema utiliza um dicionário denominado `hierarquia_sistemas`, responsável por estabelecer a ordem de 
prioridade dos equipamentos da colônia.


| Sistema | Prioridade | Classificação |
|---|---:|---|
| Suporte à vida | 1 | Essencial |
| Abastecimento de água | 2 | Essencial |
| Controle térmico | 3 | Essencial |
| Comunicação | 4 | Não essencial |
| Iluminação | 5 | Não essencial |
| Laboratório | 6 | Não essencial |

Quanto menor o número, maior a prioridade.

Essa hierarquia permite identificar quais equipamentos precisam de atenção imediata, principalmente em situações de restrição energética ou falhas operacionais.

---

## Análise e gerenciamento de energia

A análise energética é responsável por comparar a geração total de energia com o consumo registrado em cada intervalo.

A geração total considera a soma das duas fontes disponíveis:

**Geração total = geração solar + geração eólica**

Em seguida, é calculado o saldo energético:

**Saldo energético = geração total − consumo**

A partir desse resultado, o sistema identifica três situações:

- **Saldo positivo:** a geração superou o consumo, indicando excedente de energia disponível para armazenamento ou gerenciamento.
- **Saldo negativo:** o consumo foi superior à geração, indicando a necessidade de utilizar a reserva energética ou avaliar medidas de economia.
- **Saldo igual a zero:** a geração foi suficiente para atender exatamente ao consumo do intervalo.

### Reserva de energia

Para complementar a análise, foi definido um parâmetro didático de capacidade total da bateria de **500 kWh**, com uma reserva mínima de **20%**.

O percentual informado na leitura é convertido em energia armazenada:

```python
energia_armazenada = capacidade_bateria * percentual_bateria / 100
```

A energia utilizável corresponde à parcela armazenada acima da reserva mínima.

Quando existe déficit de geração, o sistema verifica se a energia utilizável é suficiente para cobrir a diferença entre geração e consumo, 
sem ultrapassar a reserva mínima configurada.

Os valores de capacidade e reserva são premissas da simulação, podendo ser ajustados conforme as especificações adotadas para o cenário.

**Observação:** a análise verifica a capacidade de cobertura do déficit de cada intervalo individualmente. O percentual da bateria informado na base 
não é atualizado automaticamente a partir do carregamento e descarregamento ao longo do dia.

---

## Monitoramento ambiental

O monitoramento ambiental verifica as condições de temperatura e vento registradas pelos sensores.

Foram estabelecidos os seguintes parâmetros para a simulação:

| Parâmetro | Condição configurada |
|---|---|
| Temperatura interna mínima | 18 °C |
| Temperatura interna máxima | 26 °C |
| Temperatura externa | Abaixo de -40 °C: alerta de frio extremo |
| Velocidade do vento | Acima de 25 m/s: alerta operacional |

A função `analisar_ambiente()` verifica se a temperatura interna está dentro do intervalo configurado, identifica situações de frio externo extremo e 
sinaliza velocidades de vento acima do limite estabelecido.

Quando existe frio extremo e o controle térmico está inativo, o sistema gera uma recomendação de verificação imediata, considerando a importância desse 
equipamento para a segurança da colônia.

---

## Identificação de padrões no histórico

A função `analisar_padroes()` percorre todas as leituras disponíveis para consolidar informações sobre o comportamento energético da colônia.

Entre os indicadores identificados estão:

- Geração total acumulada.
- Consumo total acumulado.
- Saldo energético acumulado.
- Maior consumo registrado.
- Horário em que ocorreu o maior consumo.

Essa análise permite observar o comportamento da colônia durante todo o período monitorado, identificando situações que poderiam passar despercebidas 
quando analisadas apenas as leituras individuais.

---

## Previsão de geração eólica — Regressão Linear

Para evoluir de uma análise exclusivamente reativa para uma abordagem preditiva, foi implementado um modelo de **regressão linear simples**, que utiliza a 
velocidade do vento como variável de entrada e a geração de energia eólica como variável estimada.

O treinamento é realizado uma única vez com o histórico disponível. Após o treinamento, a função `prever_geracao_eolica()` recebe uma velocidade de vento 
informada e retorna a geração estimada, sem precisar recalcular os parâmetros do modelo a cada consulta.

### Exemplo de previsão

Considerando o histórico utilizado na simulação:

**Entrada:**

```python
velocidade_vento = 11
```

**Saída aproximada:**

```text
Geração eólica estimada: 27,10 kWh
```

A previsão representa a geração estimada para uma velocidade de vento fornecida. O modelo não prevê a velocidade futura do vento.

---

## Avaliação do modelo — MAE

Para avaliar o desempenho da regressão linear, foi utilizado o Erro Absoluto Médio (MAE — Mean Absolute Error).

O indicador calcula a média das diferenças absolutas entre os valores estimados pelo modelo e os valores efetivamente registrados.

A avaliação utiliza uma abordagem cronológica: para cada estimativa, o modelo é treinado apenas com os registros anteriores à leitura avaliada. 
Dessa forma, o próprio valor que está sendo testado não participa do treinamento daquela estimativa.

---

## Tomada de decisão e recomendações

A função `tomar_decisao()` integra os resultados das análises energéticas, ambientais e operacionais para produzir mensagens de orientação.

Entre as regras implementadas estão:

- Identificar quando o consumo supera a geração.
- Verificar se a reserva disponível consegue cobrir o déficit do intervalo.
- Identificar excedentes de energia.
- Avaliar o nível percentual da bateria e recomendar atenção ou economia.
- Comparar a geração solar registrada com a geração eólica estimada.
- Identificar equipamentos inativos e destacar aqueles classificados como essenciais.
- Gerar alertas críticos quando há frio externo extremo e o controle térmico está inativo.
- Apresentar a ordem de prioridade dos equipamentos ativos.

As decisões são apresentadas como recomendações operacionais. 
---

## Análise da eficiência energética

Além das análises individuais, o sistema consolida indicadores para avaliar o comportamento energético da colônia durante as 24 horas.

São calculados:

**Cobertura da demanda:**

Representa a proporção do consumo total que foi atendida pela geração própria.

```python
cobertura_demanda = (geracao_total / consumo_total) * 100
```

Também são contabilizados:

- Quantidade de períodos com déficit de geração.
- Quantidade de períodos com excedente.
- Quantidade de períodos em equilíbrio.
- Saldo energético acumulado.

Esses indicadores permitem identificar se a geração própria foi suficiente para acompanhar a demanda energética durante o período analisado.

---

##   Resultados da simulação

Na execução da base de dados utilizada, foram obtidos os seguintes resultados:

| Indicador | Resultado |
|---|---:|
| Leituras analisadas | 24 |
| Geração acumulada | 957 kWh |
| Consumo acumulado | 1.187 kWh |
| Saldo energético | -230 kWh |
| Cobertura da demanda | 80,62% |
| Períodos com déficit | 17 |
| Períodos com excedente | 7 |
| Períodos em equilíbrio | 0 |
| Maior consumo registrado | 62 kWh |
| Horário de maior consumo | 14:00 |
| MAE da regressão linear | 0,54 kWh |

Os resultados demonstram que a geração própria não foi suficiente para atender integralmente ao consumo acumulado durante o período simulado.

A cobertura de 80,62% indica que a geração solar e eólica registrada correspondeu a aproximadamente 80,62% da demanda total. Por isso, o monitoramento 
da reserva energética e a identificação de períodos de déficit são importantes para apoiar as decisões operacionais.

---

## Visualizações

O sistema também utiliza a biblioteca Matplotlib para gerar um gráfico comparativo entre a geração eólica registrada e a geração estimada pelo modelo de regressão linear.

A visualização permite observar as diferenças entre os valores reais e estimados ao longo dos horários avaliados, facilitando a interpretação do desempenho do modelo.

---

##  Tecnologias e recursos utilizados

- **Python:** linguagem utilizada no desenvolvimento do sistema.
- **Estruturas de dados:** listas e dicionários.
- **Lógica computacional:** estruturas condicionais, funções e regras de decisão.
- **Modelagem matemática:** cálculos de geração, consumo, saldo energético e regressão linear.
- **Matplotlib:** construção de gráficos para visualização dos resultados.

---

## Como executar o projeto


### 1. Clonar o repositório

Abra o terminal e execute:

```bash
git clone URL_DO_SEU_REPOSITORIO
```

Entre na pasta do projeto:

```bash
cd NOME_DO_REPOSITORIO
```

### 3. Instalar a dependência

```bash
pip install matplotlib
```

### 4. Executar o sistema

Execute o arquivo Python disponibilizado no repositório:

```bash
python sistema_integrado.py
```

Após a execução, serão apresentados os relatórios individuais das leituras, o resumo geral da colônia, os indicadores energéticos, a avaliação da regressão 
e o gráfico comparativo de geração eólica.

---

## Organização do repositório

O repositório disponibiliza o código-fonte do sistema em formato `.py`, permitindo sua execução diretamente em um ambiente Python.

Além disso, uma versão do projeto também está sendo mantida no **Colab**, caso seja de preferência de visualização do usuário.

---

## Conclusão

O desenvolvimento do Sistema Aurora Siger permitiu integrar diferentes conceitos de programação em uma solução que simula o monitoramento e a 
tomada de decisão de uma colônia em Marte.

Por meio da organização estruturada dos dados, da aplicação de regras condicionais, da análise energética e ambiental e da utilização da regressão linear, 
o sistema consegue transformar informações simuladas em indicadores, alertas e recomendações operacionais.

A implementação demonstra como a análise de dados históricos pode complementar o monitoramento das condições atuais, contribuindo para uma abordagem 
mais preditiva e eficiente na gestão dos recursos da colônia.

Dessa forma, o projeto atende ao propósito de desenvolver um sistema computacional integrado, capaz de organizar informações, identificar situações de risco, 
estimar comportamentos e apoiar decisões relacionadas à continuidade e à eficiência das operações da missão Aurora Siger.


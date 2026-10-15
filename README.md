# Classificação de morangos: fresco ou podre

> **Onde estão os resultados?** Os arquivos gerados originalmente ficam na pasta
> `datasetmorangonovo/resultados_lmfcn_original`. A execução principal também foi copiada para
> [`resultados_lmfcn_original/strawberry_out/`](resultados_lmfcn_original/strawberry_out/) neste repositório.

Este projeto treina modelos de visão computacional para responder a uma pergunta simples:

> **O morango da foto está fresco ou podre?**

O repositório contém experimentos com **YOLO11**, **MobileNetV2** e **LMFCN + SVM**. Ele também guarda os modelos treinados, as métricas e o código usado nos testes com Raspberry Pi.

## Se você só quer ver o resultado

Não é necessário instalar nada.

- Resultado atual do LMFCN original: [`resultados_lmfcn_original/strawberry_out/resultados.json`](resultados_lmfcn_original/strawberry_out/resultados.json)
- Explicação dessa execução: [`resultados_lmfcn_original/README.md`](resultados_lmfcn_original/README.md)
- Comparação dos modelos: [`RESULTADOS_COMPARATIVOS.md`](RESULTADOS_COMPARATIVOS.md)
- Arquivos preparados para o Raspberry Pi: [`modelos_raspberry_pi/`](modelos_raspberry_pi/)

Na execução atual do LMFCN original:

- o modelo treinou com **1.200 imagens** do `setting1`: 600 frescas e 600 podres;
- a validação foi feita nas **200 imagens separadas** da pasta `test`: 100 frescas e 100 podres;
- nenhuma imagem de `test` foi usada no treinamento;
- foram usados os parâmetros do comando `strawberry_1` enviado pelo autor;
- a melhor acurácia observada em `test` foi **99,5%**, na época 5;
- o recall foi **99% para morangos frescos** e **100% para morangos podres**;
- o F1 macro foi **99,5%**.

## O que existe em cada pasta

| Caminho | Conteúdo |
|---|---|
| `codigo_jonathan/` | Código LMFCN original de Jonathan Matos, preservado como referência. |
| `executar_lmfcn_original.py` | Adaptador que executa o código original no dataset de morangos. Ele ajusta entrada, dispositivo e salvamento; não troca o algoritmo LMFCN. |
| `resultados_lmfcn_original/` | Manifestos, métricas, CNN, SVM e documentação da execução original. |
| `experimento_lmfcn/` | Implementação modular inspirada no LMFCN. Não é uma reprodução literal. |
| `testes YOLO/` | Scripts, resultados e artefatos dos experimentos com YOLO. |
| `testes MobileNetV2/` | Scripts dos experimentos com MobileNetV2. |
| `modelos_raspberry_pi/` | Modelos e scripts preparados para inferência no Raspberry Pi. |
| `resumo/` | Resumos de execuções anteriores. |

## Antes de começar

Você precisa de:

1. Python 3.9 ou mais recente;
2. Git;
3. espaço livre para instalar o PyTorch;
4. a pasta do dataset no computador.

Para confirmar se o Python está instalado, abra o PowerShell e execute:

```powershell
py --version
```

Se aparecer algo como `Python 3.12.1`, está tudo certo. Se o comando não existir, instale o Python pelo [site oficial](https://www.python.org/downloads/) e marque a opção para adicionar o Python ao `PATH`.

## Baixar este projeto

No PowerShell:

```powershell
git clone https://github.com/ChagasEng/strawberry-classification.git
cd strawberry-classification
```

O download é grande porque o repositório contém modelos e resultados de treinamento.

## Entenda os dois formatos de dataset

Este repositório trabalha com dois formatos. Eles servem para experimentos diferentes e não devem ser confundidos.

### 1. `datasetmorangonovo`: comparação e LMFCN original

Este é o formato usado no resultado de **99,5%**. A pasta pode ficar em qualquer lugar do computador, desde que você informe o caminho no comando.

```text
datasetmorangonovo/
|-- setting1/
|   |-- fresh/       # 600 imagens
|   `-- rotten/      # 600 imagens
`-- test/
    |-- fresh/       # 100 imagens
    `-- rotten/      # 100 imagens
```

O `setting1` inteiro é usado para treinamento. A pasta `test` é usada somente para validar o modelo.

### 2. `Dataset`: exemplo modular menor

Este formato é usado apenas pelo exemplo disponível em `experimento_lmfcn/`.

Baixe as imagens pela [pasta no Google Drive](https://drive.google.com/drive/folders/1d7YR44NljiyG3dp57XR8DcorJYzW8gAY), extraia o ZIP e deixe a estrutura assim:

```text
strawberry-classification/
|-- Dataset/
|   `-- Nusrat Sultana/
|       `-- Original/
|           |-- FreshStrawberry/
|           `-- RottenStrawberry/
`-- experimento_lmfcn/
```

Se aparecer `Dataset/Dataset/Nusrat Sultana`, existe uma pasta duplicada. Mova `Nusrat Sultana` para a primeira pasta `Dataset`.

As imagens de `Dataset/` não são enviadas ao GitHub porque essa pasta está no `.gitignore`.

## Reproduzir o LMFCN original

Use esta opção para executar o código original do Jonathan com o protocolo do `datasetmorangonovo`.

### Passo 1 — criar o ambiente Python

Execute na raiz do repositório:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements_lmfcn_original.txt
```

### Passo 2 — conferir o dataset

Substitua o caminho abaixo pelo local real da sua pasta:

```powershell
Get-ChildItem "C:\caminho\para\datasetmorangonovo"
```

O resultado precisa mostrar pelo menos `setting1` e `test`.

### Passo 3 — treinar e validar

```powershell
.\.venv\Scripts\python.exe .\executar_lmfcn_original.py `
  --dataset "C:\caminho\para\datasetmorangonovo" `
  --setting setting1 `
  --data-protocol external `
  --epoch 5 --relax 10 --lr 1.0 --feature 16 `
  --svclose 5 --wrclose 2 --shclose 0 `
  --width 275 --height 183 `
  --outdir ".\resultados_lmfcn_original\strawberry_out" `
  --outfile strawberry_1 `
  --batch 16 `
  --threads 4
```

Esse é o equivalente local do comando fornecido pelo autor. O caminho Linux passado originalmente em
`-i` apontava para a lista de treino dele. Aqui, `--dataset` permite que o executor crie duas listas
automaticamente: `setting1` para treino e `test` para validação. Os parâmetros omitidos no comando
continuam com os valores padrão do original: `gamma=1`, `shake=0`, `aug=1`, `norm=1` e `seed=0`.

O parâmetro `external` significa exatamente isto:

```text
setting1 completo -> treinamento
test separado     -> validação
```

Ao terminar, os arquivos serão gravados em:

```text
resultados_lmfcn_original/strawberry_out/
```

O arquivo mais importante é `resultados.json`, que contém as métricas de todas as épocas.

> **Observação:** `modelo_final.pt` representa a última época. O histórico em `resultados.json` informa qual época teve a melhor validação. Nesta execução, a última época também foi a melhor: época 5, com 99,5%.

## Executar a implementação modular

Esta versão é mais simples de modificar, mas é apenas **inspirada** no LMFCN original.

### Windows

```powershell
cd experimento_lmfcn
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m morango_lmfcn prepare --dataset ..\Dataset
.\.venv\Scripts\python.exe -m morango_lmfcn train
```

Para classificar uma imagem depois do treinamento:

```powershell
.\.venv\Scripts\python.exe -m morango_lmfcn predict "..\Dataset\Nusrat Sultana\Original\FreshStrawberry\FreshStrawberry_142.jpg"
```

O programa imprimirá `fresco` ou `podre`.

### Linux e macOS

```bash
cd experimento_lmfcn
python3 -m venv .venv
./.venv/bin/python -m pip install -r requirements.txt
./.venv/bin/python -m morango_lmfcn prepare --dataset ../Dataset
./.venv/bin/python -m morango_lmfcn train
```

## Problemas comuns

### “Python não foi encontrado”

Instale o Python e reabra o terminal. No Windows, confirme que o Python Launcher foi instalado.

### “Nenhuma imagem encontrada”

O caminho do dataset está errado ou os nomes das pastas não correspondem à estrutura mostrada acima.

### “CUDA não está disponível”

Não é um erro obrigatório. O executor usa CPU automaticamente quando não encontra uma GPU NVIDIA compatível.

### O treinamento parece parado

O LMFCN original atualiza muitos vetores de suporte individualmente. Em CPU, uma época pode passar algum tempo sem imprimir novas mensagens. Aguarde até aparecer a próxima linha `Epoch`.

### Mudei o dataset de lugar e o exemplo modular parou

Apague `experimento_lmfcn/saida/manifests/` e execute novamente o comando `prepare`. Os manifestos guardam caminhos absolutos.

## Referência

O LMFCN usado como base foi desenvolvido por Jonathan Matos: [jonathandematos/lmfcn](https://github.com/jonathandematos/lmfcn).

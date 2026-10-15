# Execução do LMFCN original

O arquivo [`executar_lmfcn_original.py`](../executar_lmfcn_original.py) carrega diretamente o LMFCN de
`codigo_jonathan/lmfcn`. Ele não substitui a rede nem o SVM: apenas monta as listas do novo dataset,
seleciona CPU ou CUDA e salva os arquivos que o código original deixava de persistir.

## Protocolo usado

- treino: todas as 1.200 imagens de `setting1` (600 `fresh` e 600 `rotten`);
- validação: somente as 200 imagens da pasta separada `test` (100 de cada classe);
- imagens de `test` usadas no treino: **zero**;
- épocas: 5;
- tamanho das imagens: 275 x 183 px;
- características: 16;
- `relax=10`, `lr=1.0`, `svclose=5`, `wrclose=2`, `shclose=0` e batch 16;
- padrões herdados do `main.py` original: `gamma=1`, `shake=0`, `aug=1`, `norm=1` e `seed=0`.

O comando original recebido foi:

```bash
python main.py --epoch 5 --relax 10 --lr 1.0 --feature 16 --svclose 5 --wrclose 2 --shclose 0 --width 275 --height 183 --outdir strawberry_out --outfile strawberry_1 --batch 16 -i /home/jonathan/git/lmfcn/strawberry/strawberry_analysis/dataset/fold1-train.txt
```

O caminho em `-i` era uma lista de imagens existente apenas no computador do autor. Para aplicar os
mesmos parâmetros ao `datasetmorangonovo` e validar na pasta `test`, foi executado o equivalente:

```powershell
.\.venv\Scripts\python.exe .\executar_lmfcn_original.py `
  --dataset "C:\caminho\para\datasetmorangonovo" `
  --setting setting1 --data-protocol external `
  --epoch 5 --relax 10 --lr 1.0 --feature 16 `
  --svclose 5 --wrclose 2 --shclose 0 `
  --width 275 --height 183 `
  --outdir ".\resultados_lmfcn_original\strawberry_out" `
  --outfile strawberry_1 --batch 16 --threads 4
```

## Resultado no conjunto `test`

| Época | Acurácia | Recall fresh | Recall rotten |
|---:|---:|---:|---:|
| 1 | 80,5% | 73% | 88% |
| 2 | 95,5% | 95% | 96% |
| 3 | 98,0% | 96% | 100% |
| 4 | 99,0% | 98% | 100% |
| 5 | **99,5%** | **99%** | **100%** |

Na melhor época, o modelo acertou **199 das 200 imagens**. A matriz de confusão é:

```text
                 previsto fresh   previsto rotten
fresh real              99                 1
rotten real              0               100
```

- acurácia: **99,5%**;
- acurácia balanceada: **99,5%**;
- F1 macro calculado pela matriz: **99,5%**.

Os arquivos dessa execução estão em [`strawberry_out/`](strawberry_out/):

- `resultados.json`: parâmetros, ambiente e métricas de todas as épocas;
- `modelo_final.pt`: pesos da CNN na época 5;
- `svm_final.joblib`: classificador SVM final;
- `referencia_final.npz`: características de referência e gamma;
- `manifests/`: listas exatas de treino e validação usadas.

## Reprodutibilidade

A execução registrada usou Python 3.12.15, PyTorch 2.14.1 CPU e scikit-learn 1.9.1. O ambiente antigo
do autor usava versões anteriores dessas bibliotecas. Portanto, o algoritmo e os parâmetros foram
preservados, mas resultados bit a bit idênticos em outro computador não são garantidos.

Uma execução anterior, com os parâmetros do exemplo genérico de imagens (`feature=2`, `relax=1`,
`lr=0.1`, 350 x 230 e batch 256), permanece em [`setting1_treino_completo/`](setting1_treino_completo/)
e atingiu 85%. Ela não é o resultado principal atual.

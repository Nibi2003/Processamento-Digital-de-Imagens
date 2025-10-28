# Processamento Digital de Imagens — Trabalho 2  
**Autora:** Beatriz Emily Silva Aguiar  
**Instituição:** Universidade Federal do Amazonas (UFAM)  
**Disciplina:** Processamento Digital de Imagens  

---

## 1. Objetivo  

O presente trabalho tem como objetivo a implementação e análise de algoritmos para **detecção de cortes de cena (Shot Boundary Detection – SBD)** e **seleção de quadros-chave (Key Frames)** em vídeos digitais.  
O sistema desenvolvido identifica automaticamente as transições entre cenas e seleciona um quadro representativo para cada segmento detectado, comparando os resultados obtidos com a rotulação manual.

---

## 2. Metodologia  

### 2.1 Rotulação Manual  
Cada vídeo foi inspecionado visualmente para identificação dos cortes de cena reais.  
Os resultados foram armazenados em arquivos `.txt`, contendo pares de índices `[start, end]` que definem as transições de cena.  
Esses dados foram utilizados como **ground truth** para avaliação dos métodos automáticos.

### 2.2 Descritores Implementados  

Foram implementados dois descritores locais, ambos baseados em particionamento mínimo de `2x2` regiões:

| Descritor | Descrição | Características |
|------------|------------|----------------|
| **Histograma Local** | Divide o frame em partições e calcula histogramas de cor (R, G, B) quantizados para cada região. Os histogramas são concatenados em um vetor global. | Rápido, simples e robusto a pequenas variações de iluminação. |
| **BIC Local (Border/Interior Classification)** | Classifica os pixels de cada partição em borda ou interior e gera histogramas separados para ambos. Os vetores resultantes são concatenados. | Sensível a mudanças estruturais e bordas de objetos. |

### 2.3 Medida de Dissimilaridade  

Para cada par de frames consecutivos `(i, i-1)`, calcula-se a diferença entre seus descritores por meio da **distância L2 entre vetores de características normalizados**:

\[
d_i = ||D_i - D_{i-1}||_2
\]

Essa medida reflete a variação global entre as distribuições de cor e textura dos quadros, não sendo uma distância espacial direta entre imagens.  
Os valores obtidos são normalizados para o intervalo `[0, 1]` de acordo com:

\[
d_i' = \frac{d_i - \min(d)}{\max(d) - \min(d)}
\]

Um corte de cena é identificado quando `d_i' > threshold`.

---

## 3. Seleção de Quadros-Chave  

Após a segmentação das cenas, é escolhido um quadro representativo (key frame) para cada trecho identificado.  
Dois critérios foram implementados:

1. **Método "middle":** seleciona o frame central da cena.  
2. **Método "max_diff":** seleciona o frame com maior diferença em relação ao anterior dentro do intervalo da cena.  

Quadros excessivamente claros ou escuros são descartados com base em seu brilho médio (`< 30` ou `> 220`).  
O algoritmo tenta substituir quadros inválidos por frames vizinhos até ±2 posições.

---

## 4. Métricas de Avaliação  

O desempenho dos algoritmos foi avaliado pelas seguintes métricas:

| Métrica | Definição | Descrição |
|----------|------------|-----------|
| **Precisão (P)** | TP / (TP + FP) | Proporção de cortes detectados corretamente. |
| **Recall (R)** | TP / (TP + FN) | Proporção de cortes reais detectados. |
| **F1-score** | 2PR / (P + R) | Média harmônica entre precisão e recall. |
| **Acurácia (professor)** | (acertos ±1s) / total | Considera 1 segundo de tolerância em relação ao ground truth. |

---

## 5. Resultados  

Foram testados diferentes valores de limiar (`threshold`) para avaliar a sensibilidade dos métodos.  

Observações gerais:
- Thresholds baixos (0.1–0.3) resultam em maior número de cortes detectados e mais falsos positivos.  
- Thresholds altos (0.5–0.7) reduzem falsos positivos, mas podem omitir transições sutis.  
- O BIC Local apresentou maior precisão em vídeos com bordas e texturas evidentes.  
- O Histograma Local foi mais robusto em vídeos de animação e clipes musicais.

Exemplo de resultados obtidos:

| Vídeo | Método | Threshold | Precisão | Recall | F1-score |
|--------|---------|------------|-----------|----------|-----------|
| Anya_Desenho.mp4 | BIC Local | 0.3 | 0.769 | 1.000 | 0.870 |
| Anya_Desenho.mp4 | Histograma Local | 0.3 | 0.650 | 1.000 | 0.788 |
| Anya_Desenho.mp4 | BIC Local | 0.5 | 0.895 | 0.850 | 0.872 |

---

## 6. Visualização  

Para análise qualitativa, foram implementadas funções de visualização que exibem os frames dos vídeos em linhas contínuas:

- **Linha 1:** Rótulos manuais (ground truth)  
- **Linha 2:** Cortes detectados pelo método BIC Local  
- **Linha 3:** Cortes detectados pelo método Histograma Local  

Os quadros identificados como **key frames** são destacados em vermelho.  
Cada linha representa a sequência temporal dos frames do vídeo, permitindo a comparação direta entre a detecção manual e automática.

---

## 7. Organização do Repositório  

| Arquivo | Descrição |
|----------|------------|
| `OFFICIAL_trab2.ipynb` | Versão final e organizada do código, pronta para execução no Google Colab. |
| `rascunho_trab2.py` | Versão preliminar contendo o raciocínio, testes e rotulação manual. |

- **Recomendações** Não rode o rascunho, apenas veja as saídas que ja foram rodadas
- **DATASET** O download do dataset ja esta incluído no código OFFICIAL


---

## 8. Execução no Google Colab  

1. Montar o Google Drive:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')

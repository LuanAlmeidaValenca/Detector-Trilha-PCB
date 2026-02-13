# 🔬 Identificação e Rastreamento de Trilhas em PCB

> Trabalho Final — Disciplina de **Processamento de Imagens**

**Autores:** Gabriel Batista Barbosa · Luan Almeida Valença

---

## Índice

- [Resumo](#resumo)
- [Objetivo](#objetivo)
- [Pipeline de Processamento](#pipeline-de-processamento)
  - [1. Pré-processamento e Binarização](#1-pré-processamento-e-binarização)
  - [2. Detecção de Pads (Hole-First)](#2-detecção-de-pads-hole-first)
  - [3. Esqueletização e Mapeamento de Trilhas](#3-esqueletização-e-mapeamento-de-trilhas)
  - [4. Geração da Netlist](#4-geração-da-netlist)
- [Tecnologias e Bibliotecas](#tecnologias-e-bibliotecas)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Como Executar](#como-executar)
- [Parâmetros Ajustáveis](#parâmetros-ajustáveis)
- [Resultados Esperados](#resultados-esperados)
- [Referências Técnicas](#referências-técnicas)

---

## Resumo

Este projeto implementa uma solução computacional para a **engenharia reversa de conexões elétricas em Placas de Circuito Impresso (PCB)** a partir de imagens digitalizadas. O sistema identifica automaticamente os pontos de solda (_pads_), mapeia as trilhas condutoras de cobre e gera uma **netlist** — a lista completa de conexões elétricas entre os componentes da placa.

A abordagem é inteiramente baseada em técnicas clássicas de processamento de imagens, sem uso de aprendizado de máquina, utilizando um pipeline sequencial de binarização adaptativa, detecção morfológica, esqueletização e cruzamento espacial.

---

## Objetivo

Dada uma imagem digital de uma PCB, o sistema deve:

1. **Segmentar** as trilhas de cobre do substrato da placa.
2. **Detectar** todos os pads (ilhas de solda / furos de passagem).
3. **Rastrear** as trilhas condutoras que interligam os pads.
4. **Gerar** a netlist no formato `Pad_A <-> Pad_B`, indicando quais pads estão eletricamente conectados.

---

## Pipeline de Processamento

O processamento é organizado em quatro etapas principais:

### 1. Pré-processamento e Binarização

| Etapa                | Técnica                                                    |
| -------------------- | ---------------------------------------------------------- |
| Carregamento         | Leitura da imagem (suporte a RGB, RGBA e escala de cinza)  |
| Conversão            | Transformação para escala de cinza                         |
| Binarização          | **Limiarização adaptativa de Sauvola**                     |
| Ajuste de polaridade | Garantia de que o fundo é preto (0) e o cobre é branco (1) |

**Por que Sauvola?**
A binarização de Sauvola é uma técnica de limiarização local que calcula um threshold diferente para cada pixel com base na média e no desvio padrão de sua vizinhança. Isso a torna ideal para imagens de PCB, onde:

- A iluminação pode ser desigual.
- As trilhas de cobre possuem espessuras variadas.
- Os detalhes finos (trilhas estreitas) precisam ser preservados sem gerar curto-circuitos artificiais na binarização.

O parâmetro `k` controla a sensibilidade: valores menores preservam mais cobre (trilhas grossas), enquanto valores maiores permitem que os furos dos pads fiquem visíveis na imagem binária.

### 2. Detecção de Pads (Hole-First)

A estratégia **Hole-First** identifica os pads através de seus furos centrais, em vez de buscar formas circulares diretamente:

1. **Limpeza de bordas:** Remove artefatos nas margens da imagem (margem de 5 pixels).
2. **Preenchimento de buracos:** Aplica `binary_fill_holes` para gerar uma versão sólida da imagem.
3. **Extração de furos:** A diferença entre a versão sólida e a original revela os furos dos pads.
4. **Filtragem por área:** Remove ruídos menores que 3 pixels com `remove_small_objects`.
5. **Rotulagem e centroides:** Cada furo é rotulado e seu centroide é calculado como coordenada do pad.
6. **Ordenação:** Os pads são ordenados por posição (Y, X) para identificação consistente.

### 3. Esqueletização e Mapeamento de Trilhas

| Etapa          | Descrição                                                                    |
| -------------- | ---------------------------------------------------------------------------- |
| Esqueletização | Redução morfológica das trilhas de cobre a linhas de 1 pixel de espessura    |
| Rotulagem      | Cada segmento contínuo do esqueleto recebe um rótulo único (conectividade 8) |

A **esqueletização** (ou afinamento) preserva a topologia das trilhas enquanto elimina a largura, permitindo mapear os caminhos condutores como grafos de conectividade.

### 4. Geração da Netlist

O cruzamento espacial entre pads e trilhas funciona da seguinte forma:

1. Para cada pad detectado, uma **janela de busca** (raio de 5 pixels) é definida ao redor de seu centroide.
2. Dentro dessa janela, verifica-se quais segmentos do esqueleto rotulado estão presentes.
3. Se um mesmo segmento de esqueleto toca **dois ou mais pads**, eles estão eletricamente conectados.
4. Todas as combinações de pares conectados são geradas e formatadas como `Pad_A <-> Pad_B`.

---

## Tecnologias e Bibliotecas

| Biblioteca       | Versão Mínima | Função                                                         |
| ---------------- | ------------- | -------------------------------------------------------------- |
| **Python**       | 3.8+          | Linguagem base                                                 |
| **NumPy**        | 1.21+         | Manipulação de arrays e operações matriciais                   |
| **Matplotlib**   | 3.4+          | Visualização de imagens e gráficos intermediários              |
| **Scikit-Image** | 0.18+         | Binarização (Sauvola), morfologia, esqueletização, rotulagem   |
| **SciPy**        | 1.7+          | Transformada de distância euclidiana, preenchimento de buracos |

---

## Estrutura do Projeto

```
trabalho-final/
├── main.ipynb          # Notebook principal com todo o pipeline
├── Imagens/            # Pasta com as imagens de PCB para processamento
│   └── img1.png        # Imagem de entrada (PCB digitalizada)
└── README.md           # Este arquivo
```

---

## Como Executar

### Pré-requisitos

```bash
pip install numpy matplotlib scikit-image scipy
```

### Execução

1. Clone o repositório:

   ```bash
   git clone https://github.com/LuanAlmeidaValenca/Detector-Trilha-PCB.git
   cd Detector-Trilha-PCB
   ```

2. Coloque a imagem da PCB na pasta `Imagens/` com o nome `img1.png` (ou altere a variável `NOME_ARQUIVO` na célula 3 do notebook).

3. Abra e execute o notebook:

   ```bash
   jupyter notebook main.ipynb
   ```

4. Execute as células sequencialmente (Células 2 → 3 → 4 → 5 → 6).

---

## Parâmetros Ajustáveis

Os principais parâmetros que podem ser calibrados conforme a imagem de entrada:

| Parâmetro      | Localização | Padrão       | Descrição                                                             |
| -------------- | ----------- | ------------ | --------------------------------------------------------------------- |
| `NOME_ARQUIVO` | Célula 3    | `'img1.png'` | Nome do arquivo de imagem na pasta `Imagens/`                         |
| `window_size`  | Célula 3    | `25`         | Tamanho da janela local para binarização Sauvola (px)                 |
| `k_factor`     | Célula 3    | `0.2`        | Sensibilidade da binarização (`0.05`=grosso, `0.2`=médio, `0.5`=fino) |
| `margem`       | Célula 4    | `5`          | Margem de segurança para limpeza de bordas (px)                       |
| `min_size`     | Célula 4    | `3`          | Área mínima (px) para considerar um furo como pad válido              |
| `WIN`          | Célula 6    | `5`          | Raio da janela de busca no cruzamento espacial pad–esqueleto (px)     |

---

## Resultados Esperados

Ao final da execução, o sistema produz:

- **Imagem binarizada** com as trilhas de cobre segmentadas.
- **Mapa de pads** com identificação numérica de cada ilha de solda detectada.
- **Esqueleto sobreposto** mostrando o caminho central de cada trilha.
- **Netlist textual** no formato:
  ```
  ========================================
  NETLIST FINAL (N Conexões)
  ========================================
  1 <-> 5
  2 <-> 7
  3 <-> 12
  ...
  ```

---

## Referências Técnicas

- **Sauvola, J. & Pietikäinen, M.** (2000). _Adaptive document image binarization_. Pattern Recognition, 33(2), 225–236.
- **Zhang, T. Y. & Suen, C. Y.** (1984). _A fast parallel algorithm for thinning digital patterns_. Communications of the ACM, 27(3), 236–239.
- **Scikit-Image Documentation** — [scikit-image.org](https://scikit-image.org/)
- **SciPy ndimage** — [docs.scipy.org/doc/scipy/reference/ndimage.html](https://docs.scipy.org/doc/scipy/reference/ndimage.html)

---

<p align="center">
  Desenvolvido como Trabalho Final para a disciplina de <strong>Processamento de Imagens</strong>.
</p>

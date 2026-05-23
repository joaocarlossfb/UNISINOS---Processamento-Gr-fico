# Parallax Scrolling — Processamento Gráfico

Projeto desenvolvido para a disciplina de Processamento Gráfico (Unisinos).  
Implementa um efeito de parallax scrolling com personagem móvel usando OpenGL moderno (core profile 4.1).

---

## Dependências

| Biblioteca | Finalidade |
|------------|-----------|
| GLFW 3     | Criação de janela e captura de input |
| GLEW       | Carregamento das extensões OpenGL |
| GLM        | Matemática vetorial/matricial |
| stb_image  | Carregamento de imagens PNG/JPG |

---

## Estrutura de assets esperada

```
assets/
  layers/
    layer1.png   ← fundo mais distante (ex: céu)
    layer2.png   ← segundo plano (ex: nuvens)
    layer3.png   ← plano intermediário (ex: vegetação distante)
    layer4.png   ← primeiro plano (ex: vegetação próxima)
  player/
    player.png   ← sprite do personagem (PNG com transparência)
```

As imagens de camada devem ser tileable horizontalmente para que o scroll funcione sem costuras visíveis.

---

## Compilação

```bash
g++ main.cpp -o parallax \
    -lGL -lGLEW -lglfw \
    -std=c++17
```

No macOS, substituir `-lGL` por `-framework OpenGL`.

---

## Controles

| Tecla | Ação |
|-------|------|
| `←` `→` | Move o personagem (aciona o scroll do cenário) |
| `↑` `↓` | Move o personagem verticalmente |
| `ESC` | Fecha a janela |

---

## Como o parallax foi implementado

O efeito é baseado em **deslocamento de coordenadas UV** no vertex shader, não em movimentação de geometria.

Cada camada é um quad estático que ocupa a tela inteira. Ao pressionar uma tecla, o valor `worldOffset` é acumulado. Na hora de renderizar, cada camada recebe um offset UV calculado assim:

```
uvOffX = (worldOffset.x × speedFactor) / SCREEN_W
```

Esse valor é somado às coordenadas UV no shader, fazendo a textura "deslizar" por baixo do quad. Como as texturas estão configuradas com `GL_REPEAT`, o scroll é infinito sem mostrar bordas.

Cada camada tem um `speedFactor` diferente:

| Camada | speedFactor | Comportamento |
|--------|-------------|---------------|
| layer1 | 0.02 | Quase estática — sensação de grande distância |
| layer2 | 0.20 | Lenta |
| layer3 | 0.55 | Velocidade média |
| layer4 | 1.20 | Mais rápida que o personagem — dá sensação de proximidade |

O personagem permanece fixo no centro horizontal da tela. A ilusão de deslocamento vem inteiramente das camadas se movendo atrás dele, em velocidades diferentes.

---

## Detalhes técnicos relevantes

- **Projeção ortográfica pixel-perfect**: origem no canto superior esquerdo, 1 unidade = 1 pixel.
- **Transparência do sprite**: fragment shader do personagem descarta pixels com `alpha < 0.5` via `discard`.
- **Delta time**: todo movimento é multiplicado pelo tempo entre frames (`dt`), garantindo velocidade consistente independente do FPS.
- **Ordem de renderização**: profundidade controlada pela ordem de desenho — sem Z-buffer.

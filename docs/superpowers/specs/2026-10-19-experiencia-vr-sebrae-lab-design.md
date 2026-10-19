# Missão Estágio: experiência VR do Sebrae Lab

Desenho aprovado em 19/10/2026. Entrega da P2 em 14/11/2026.

## Contexto

Projeto do grupo Rush B para a disciplina de Tecnologias Emergentes (UNDB, 5º período), desafio "Visita Guiada". O jogador é um estagiário no primeiro dia no Sebrae Lab: encontra o gerente, cumpre pequenas missões pelo espaço e, ao fim do expediente, libera a sala de descompressão com boliche e arco e flecha.

A P2 avalia cinco interações:

1. Pegar itens de inovação nas mesas
2. Interagir com os murais informativos
3. Conversar com o funcionário (NPC)
4. Jogar boliche
5. Atirar com arco e flecha

As missões (porta, café, arrumação do coworking) dão a narrativa, mas não estão na lista avaliada. Por isso a ordem de construção prioriza as cinco interações e deixa a narrativa por cima delas.

## Restrições

- Unity **6000.3.14f1** (Unity 6.3 LTS), exigida pelo grupo.
- Alvo: **Meta Quest 2 ou 3**, com build Android instalado no óculos. O notebook de desenvolvimento tem GPU integrada (Intel Iris Xe) e não suporta Quest Link, então os testes diários usam o XR Device Simulator.
- Duas pessoas editam o projeto, nunca ao mesmo tempo.
- Assets obrigatórios: `SebraeLabUnityAssets.unitypackage` (27 modelos FBX do Sebrae Lab e 4 texturas). Modelos externos são permitidos para o que falta.
- Disco do notebook apertado (cerca de 7 GB livres após instalar a Unity).

## Parte 1: Fundação

**Local.** Repositório `Nybly/tecnologias-emergentes2`, clonado fora do OneDrive. A pasta `Library` da Unity é reescrita o tempo todo e a sincronização do OneDrive trava arquivos e consome espaço.

**Criação do projeto.**

1. Projeto "Visita Guiada" criado no Unity Hub com o template de VR na 6000.3.14f1. Traz XR Interaction Toolkit 3.5.1 com Starter Assets e Hands Interaction Demo, XR Hands, Meta OpenXR com os perfis de controle do Quest, `BasicScene` e `SampleScene`. Já está na `main` do repositório.
2. Importar o XR Device Simulator pelo Package Manager (sample do XR Interaction Toolkit).
3. Conferir a configuração de Quest e baixar o Android mínimo para a API 32, para cobrir o Quest 2.
4. Importar `SebraeLabUnityAssets.unitypackage`.
5. Criar a cena `SebraeLab` a partir de uma cópia da `BasicScene`.

**Mantido como está.** URP (pipeline indicada para o Quest), XR Interaction Toolkit 3.5.1 e o `.gitignore` padrão da Unity que já está no repositório. Sem Git LFS: os modelos somam cerca de 10 MB.

**Fluxo de trabalho.** Pull antes de abrir a Unity, commit e push ao fechar, aviso no grupo ao começar e ao terminar.

**Testes.** Dia a dia no XR Device Simulator (teclado e mouse). No óculos, build Android por cabo USB com o Quest em modo desenvolvedor, ativado pelo app Meta Horizon com conta de desenvolvedor.

## Parte 2: Organização

**Cenas.**

| Cena | Uso |
|---|---|
| `SebraeLab.unity` | Cena principal, com a experiência inteira |
| `SampleScene.unity` | Cena de tutorial do template, sem alteração |
| `BasicScene.unity` | Cena mínima do template (jogador, EventSystem e teleporte), base da `SebraeLab`. Os exemplos do Toolkit ficam em `Assets/Samples/` como referência |

**Hierarquia da `SebraeLab`.**

```
SebraeLab
├── _Ambiente          prédio e móveis do pacote, iluminação
├── _XR                XR Origin, XR Interaction Manager, Device Simulator
├── _UI                boas-vindas, objetivo atual, "Expediente Concluído"
├── _Audio             música ambiente
├── Interacoes
│   ├── ItensDeInovacao
│   ├── MuralDoConhecimento
│   ├── NPC_Gerente
│   ├── Boliche
│   └── ArcoEFlecha
└── Missoes            gerenciador, porta, café, coworking
```

Cada interação é um prefab completo. Ajustes são feitos no prefab, e a cena principal só posiciona os prefabs.

**Pastas.**

```
Assets/
├── SebraeLab/         conteúdo do grupo
│   ├── Scenes/
│   ├── Prefabs/
│   ├── Scripts/
│   ├── Dialogos/
│   ├── Materiais/
│   ├── Modelos/       modelos externos (NPC, copo, livros...)
│   └── Audio/
├── Sebraelab3D/       pacote oficial, somente leitura
├── Samples/           XR Interaction Toolkit, somente leitura
└── XRI/
```

Nada em `Sebraelab3D/` ou `Samples/` é editado. Para alterar um material ou modelo de lá, duplicar para `SebraeLab/` e editar a cópia.

## Parte 3: As cinco interações

Ordem de construção: 1, 2, 3, 4, 5, da mais simples para a mais difícil.

### 1. Pegar itens de inovação

- XR Toolkit: `XR Grab Interactable` e `Rigidbody` em cada item.
- Script `ItemDeInovacao.cs`: na primeira vez que o item é pego, mostra uma etiqueta flutuante com nome e descrição curta.
- Pronto quando: três itens podem ser pegos, girados e soltos, e cada um mostra sua etiqueta.

### 2. Mural do Conhecimento

- XR Toolkit: `XR Simple Interactable` no quadro, acionado por raio e gatilho.
- Script `PainelInformativo.cs`: abre um painel de UI em world space diante do jogador, com o texto sobre o hub e o botão "Entendido", que fecha o painel.
- Pronto quando: apontar, abrir, ler e fechar com "Entendido".

### 3. Conversa com o NPC gerente

- `Dialogo.cs` (ScriptableObject): lista de falas editável no Inspector.
- `SistemaDeConversa.cs`: com o jogador por perto, mostra o botão "Conversar". Exibe as falas uma a uma com "Próximo" e dispara um evento ao terminar.
- Modelo: personagem gratuito do Mixamo com animação idle. Sem locomoção e sem voz.
- Pronto quando: aproximar, conversar, percorrer todas as falas e o diálogo fechar.

### 4. Boliche

- XR Toolkit: bola com `XR Grab Interactable` e arremesso ao soltar (throw on detach).
- Física: pinos com `Rigidbody`, pista com atrito baixo.
- Script `Boliche.cs`: conta pinos derrubados pela inclinação, mostra o placar e reposiciona pinos e bola no botão "Reiniciar".
- Modelos: primitivas (esfera, cilindros, plano) com materiais próprios.
- Pronto quando: arremessar, derrubar pinos, placar correto e reinício funcionando.

### 5. Arco e flecha

O XR Interaction Toolkit não tem arco pronto.

- XR Toolkit: arco como `XR Grab Interactable` em uma mão. A corda é outro interactable, puxado pela outra mão.
- `Arco.cs`: mede a tração da corda de 0 a 1, estica a corda visual e, ao soltar, lança a flecha com força proporcional à tração.
- `Flecha.cs`: voo com física, orientação acompanhando a velocidade, crava no que atingir.
- `Alvo.cs`: pontua por anel (centro vale mais) e exibe a pontuação.
- No simulador o uso com duas mãos é desajeitado. A validação final é no Quest.
- Pronto quando: pegar o arco, puxar, soltar, a flecha seguir trajetória curva, cravar no alvo e pontuar.

## Parte 4: Narrativa

**Gerenciador de missões.** `GerenciadorDeMissoes.cs` mantém a lista ordenada de objetivos e mostra o atual num painel preso à visão. As interações não conhecem o gerenciador: apenas disparam eventos (diálogo terminado, mural fechado, item pego). O gerenciador escuta esses eventos e avança. Assim cada interação continua testável sozinha.

**Roteiro.**

| Momento | Objetivo exibido | Conclusão | Base |
|---|---|---|---|
| Início, na recepção | Boas-vindas e música | Botão "Começar" | Painel da interação 2 |
| Missão 1 | Encontre o gerente | Diálogo concluído | Interação 3 |
| Missão 2 | Ache sua sala | Abrir a porta pela maçaneta | `Hinge Joint` |
| Missão 3 | Leia o Mural do Conhecimento | "Entendido" | Interação 2 |
| Missão 4 | Pegue um café na copa | Agarrar o copo | Interação 1 |
| Missão 5 | Organize o coworking | Livros e notebooks nos encaixes | `XR Socket Interactor` |
| Fim | Expediente Concluído | Sala de descompressão liberada | Interações 4 e 5 |

## Cronograma

| Semana | Entrega |
|---|---|
| 19 a 25/10 | Parte 1 completa, cena do Sebrae Lab montada, primeiro build no Quest, interações 1 e 2 |
| 26/10 a 1/11 | Interações 3 e 4 |
| 2 a 8/11 | Interação 5, gerenciador de missões, missões 1, 3 e 4 |
| 9 a 14/11 | Missões 2 e 5, áudio, polimento, testes no Quest, ensaio |

Se o prazo apertar, cortar nesta ordem: missão 5 (coworking), depois missão 2 (porta). As cinco interações avaliadas não entram no corte.

## Fora do escopo

- NPC com locomoção ou voz
- Multiplayer, salvamento de progresso, menu de opções
- Mais de uma cena de jogo

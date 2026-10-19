# Semana 1: fundação, cena do Sebrae Lab e interações 1 e 2

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Projeto Unity rodando no Quest com a cena do Sebrae Lab navegável, itens de inovação que podem ser pegos e o Mural do Conhecimento funcionando.

**Architecture:** O projeto parte de uma cópia do "My project" (XR Interaction Toolkit 3.0.11 com Starter Assets e Device Simulator), convertida para a Unity 6000.3.14f1 e configurada para Quest via OpenXR. O código do grupo fica no assembly `SebraeLab` (`Assets/SebraeLab/Scripts`). Cada interação é um MonoBehaviour pequeno que escuta eventos do XR Toolkit e expõe `UnityEvent` para a camada de missões das semanas seguintes.

**Tech Stack:** Unity 6000.3.14f1, URP, XR Interaction Toolkit 3.0.11, OpenXR com Meta Quest Support, TextMeshPro, Unity Test Framework (NUnit, EditMode).

Spec: `docs/superpowers/specs/2026-10-19-experiencia-vr-sebrae-lab-design.md`

## Global Constraints

- Versão da Unity: **6000.3.14f1**. Não abrir o projeto em outra versão.
- Alvo: Meta Quest 2 e 3, plataforma Android, OpenXR.
- XR Interaction Toolkit fica em **3.0.11**.
- Nada em `Assets/Sebraelab3D/` ou `Assets/Samples/` é editado. Alterações vão em cópias dentro de `Assets/SebraeLab/`. Exceção: configurações de importação dos FBX (Scale Factor, Generate Colliders), que só mudam o `.meta`.
- Código do grupo no namespace `SebraeLab`, nomes em português.
- Repositório: `Nybly/tecnologias-emergentes2`, clone local em `C:\Users\Gilberto Jr\Downloads\Claude\Projetos\tecnologias-emergentes` (fora do OneDrive).
- Pull antes de abrir a Unity, commit ao fechar. Push só com autorização do Gilberto.
- Mensagens de commit seguem `CLAUDE.md` do workspace: imperativo, sem travessão, sem assinatura de ferramenta.

## Quem faz o quê

Cada tarefa indica o executor:

- **[Agente]**: arquivos, scripts e testes. Roda sem a Unity aberta.
- **[Gilberto]**: passos dentro do editor da Unity ou no óculos. O agente não clica na Unity. Cada passo explica o motivo, para fins de aprendizado.

## Rodar os testes (referência para todas as tarefas)

Com o editor da Unity **fechado** (o modo batch não abre um projeto que já está aberto):

```bash
"/c/Program Files/Unity/Hub/Editor/6000.3.14f1/Editor/Unity.exe" -batchmode -projectPath "C:/Users/Gilberto Jr/Downloads/Claude/Projetos/te2" -runTests -testPlatform EditMode -testResults "C:/Users/Gilberto Jr/Downloads/Claude/Projetos/te-test-results.xml" -logFile -
```

Sem `-quit`: o `-runTests` encerra sozinho. O resultado fica no XML, fora do repositório. Para ler o resumo:

```bash
grep -o '<test-run[^>]*>' "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te-test-results.xml"
```

`result="Passed"` e `failed="0"` indicam sucesso.

## Mapa de arquivos

| Arquivo | Responsabilidade |
|---|---|
| `Assets/SebraeLab/Scripts/SebraeLab.asmdef` | Assembly do código do grupo |
| `Assets/SebraeLab/Scripts/Interacoes/ItemDeInovacao.cs` | Interação 1: etiqueta na primeira pegada, evento para missões |
| `Assets/SebraeLab/Scripts/Interacoes/VirarParaCamera.cs` | Mantém etiquetas viradas para o jogador |
| `Assets/SebraeLab/Scripts/Interacoes/PainelInformativo.cs` | Interação 2: abre e fecha painel diante do jogador |
| `Assets/SebraeLab/Tests/EditMode/SebraeLab.Tests.EditMode.asmdef` | Assembly de testes |
| `Assets/SebraeLab/Tests/EditMode/ItemDeInovacaoTests.cs` | Testes da interação 1 |
| `Assets/SebraeLab/Tests/EditMode/PainelInformativoTests.cs` | Testes da interação 2 |
| `Assets/SebraeLab/Scenes/SebraeLab.unity` | Cena principal |
| `Assets/SebraeLab/Prefabs/ItemDeInovacao_*.prefab` | Itens das mesas |
| `Assets/SebraeLab/Prefabs/MuralDoConhecimento.prefab` | Mural com painel |

---

### Task 1: Montar o projeto no repositório [Agente]

**Files:**
- Create: `Assets/`, `Packages/`, `ProjectSettings/` (cópia do My project)

**Interfaces:**
- Consumes: projeto em `C:\Users\Gilberto Jr\OneDrive - aluno.undb.edu.br\Slides e outras coisas UNDB\5º Período\Tecnologias Emergentes\My project`
- Produces: projeto Unity na raiz do repositório, ainda na versão 6000.0.83f1

- [ ] **Step 1: Copiar as três pastas e o `.vsconfig`**

```bash
SRC="/c/Users/Gilberto Jr/OneDrive - aluno.undb.edu.br/Slides e outras coisas UNDB/5º Período/Tecnologias Emergentes/My project"
DST="/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
cp -r "$SRC/Assets" "$SRC/Packages" "$SRC/ProjectSettings" "$SRC/.vsconfig" "$DST/"
```

- [ ] **Step 2: Conferir que nada gerado entrou no stage**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A && git status --short | wc -l
git diff --cached --name-only | grep -E '^(Library|Temp|Logs|UserSettings|obj)/' | wc -l
git diff --cached --name-only | grep -E '\.(csproj|sln)$' | wc -l
```

Expected: cerca de 500 arquivos no stage, e `0` nas duas últimas contagens.

- [ ] **Step 3: Varrer segredos**

```bash
git grep --cached -IlEi '(api[_-]?key|ghp_|gho_|sk-[a-z0-9]{20}|AKIA[0-9A-Z]{16}|BEGIN (RSA|PRIVATE))' -- . ':!*.meta'
```

Expected: nenhuma saída. Se algo aparecer, ler o arquivo antes de seguir.

- [ ] **Step 4: Commit**

```bash
git commit -m "Adiciona projeto Unity base com XR Interaction Toolkit" -m "Copiado do My project (Unity 6.0) com SampleScene, DemoScene, Starter Assets e XR Device Simulator. A conversão para a 6000.3.14f1 vem no próximo commit."
```

---

### Task 2: Converter para a Unity 6000.3.14f1 [Gilberto]

**Files:**
- Modify: `ProjectSettings/ProjectVersion.txt`, `Packages/manifest.json`, `Packages/packages-lock.json` e arquivos `.meta` gerados pela Unity

**Interfaces:**
- Consumes: Task 1
- Produces: projeto abrindo sem erros na 6000.3.14f1

- [ ] **Step 1: Adicionar o projeto no Unity Hub**

Hub › **Projects** › **Add** › **Add project from disk** › escolher a pasta `te2`. Na coluna **Editor Version**, selecionar **6000.3.14f1**.

*Por quê:* o Hub registra o projeto e qual editor abre. A pasta veio da 6.0, então o Hub vai perguntar se pode converter.

- [ ] **Step 2: Abrir e aceitar a conversão**

Clicar no projeto. Na janela "Change Editor version?", confirmar **Change Version**. Se aparecer aviso de atualização de pacotes (API Updater), aceitar.

*Por quê:* a 6.3 reescreve alguns arquivos de configuração no formato novo. É uma via de mão única, por isso o projeto não deve mais ser aberto na 6.0. A primeira abertura demora, porque a pasta `Library` é criada do zero.

- [ ] **Step 3: Conferir o Console**

Menu **Window › General › Console**. Esperado: nenhum erro vermelho. Avisos amarelos dos exemplos podem ser ignorados. Se houver erro vermelho, copiar o texto e passar ao agente antes de continuar.

- [ ] **Step 4: Testar o rig no simulador**

Abrir `Assets/Scenes/SampleScene`. Se a cena não tiver o rig, arrastar para a Hierarchy:
- `Assets/Samples/XR Interaction Toolkit/3.0.11/Starter Assets/Prefabs/XR Origin (XR Rig).prefab`
- `Assets/Samples/XR Interaction Toolkit/3.0.11/XR Device Simulator/XR Device Simulator.prefab`

Apertar **Play**. Esperado: a câmera fica na altura de uma pessoa e o painel do simulador aparece no canto da tela com os atalhos de teclado. Testar andar e girar. Sair do Play **sem salvar** a cena.

*Por quê:* confirma que XR Toolkit e simulador sobreviveram à conversão antes de empilhar mais coisas.

- [ ] **Step 5: Fechar a Unity e verificar a versão [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
head -1 ProjectSettings/ProjectVersion.txt
git status --short | grep -E '^(\?\?| M) (Library|Temp|Logs|UserSettings)/' | wc -l
```

Expected: `m_EditorVersion: 6000.3.14f1` e `0`.

- [ ] **Step 6: Commit [Agente]**

```bash
git add -A
git commit -m "Converte o projeto para a Unity 6000.3.14f1"
```

---

### Task 3: Configurar OpenXR para o Meta Quest [Gilberto]

**Files:**
- Modify: `Packages/manifest.json`, `ProjectSettings/ProjectSettings.asset`, `ProjectSettings/EditorBuildSettings.asset`
- Create: `Assets/XR/` (configurações de XR Plugin Management e OpenXR)

**Interfaces:**
- Consumes: Task 2
- Produces: projeto na plataforma Android com OpenXR e Meta Quest Support ativos

- [ ] **Step 1: Trocar a plataforma para Android**

**File › Build Profiles** › **Android** › **Switch Platform**.

*Por quê:* o Quest roda Android. Trocar agora faz a Unity reimportar texturas no formato do óculos uma vez só. Demora alguns minutos.

- [ ] **Step 2: Instalar o XR Plugin Management**

**Edit › Project Settings › XR Plug-in Management** › **Install XR Plugin Management**.

*Por quê:* é o gerenciador que liga o "motor" de VR. Sem ele, o XR Toolkit não recebe dados de cabeça e controles.

- [ ] **Step 3: Ativar OpenXR com Meta Quest**

Ainda em **XR Plug-in Management**, aba com o ícone do Android: marcar **OpenXR** e, logo abaixo, o grupo **Meta Quest feature group**. Aceitar a instalação do pacote OpenXR se a Unity pedir.

Depois, aba **Windows** (ícone de monitor): marcar **OpenXR** também.

*Por quê:* OpenXR é o padrão aberto que o Quest entende. O grupo Meta Quest ativa os recursos específicos do óculos. Ativar no Windows permite que o simulador e um eventual óculos ligado ao PC usem a mesma configuração.

- [ ] **Step 4: Perfis de controle**

**XR Plug-in Management › OpenXR**, aba Android. Em **Enabled Interaction Profiles**, clicar em **+** e adicionar:
- **Oculus Touch Controller Profile** (Quest 2)
- **Meta Quest Touch Plus Controller Profile** (Quest 3)

Repetir na aba Windows.

*Por quê:* sem perfil de controle o OpenXR não sabe mapear os botões, e os controles ficam mortos no óculos.

- [ ] **Step 5: Resolver o Project Validation**

**XR Plug-in Management › Project Validation**, aba Android › **Fix All**. Repetir até a lista ficar sem erros vermelhos.

Conferir em **Project Settings › Player › Android › Other Settings**:
- **Scripting Backend**: IL2CPP
- **Target Architectures**: só ARM64
- **Minimum API Level**: Android 12L (API level 32)

*Por quê:* o Quest exige ARM64 e IL2CPP. O validador da Unity conhece essas regras e corrige quase tudo sozinho.

- [ ] **Step 6: Testar no simulador de novo**

Abrir a `SampleScene` e dar **Play**. Esperado: o mesmo comportamento da Task 2, sem erros novos no Console.

- [ ] **Step 7: Fechar a Unity e verificar os pacotes [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
grep -E '"com.unity.xr.(management|openxr)"' Packages/manifest.json
grep -E 'AndroidTargetArchitectures|scriptingBackend' -A2 ProjectSettings/ProjectSettings.asset | head -8
```

Expected: as duas linhas de pacote presentes; `AndroidTargetArchitectures: 2` (ARM64) e `Android: 1` em `scriptingBackend` (IL2CPP).

- [ ] **Step 8: Commit [Agente]**

```bash
git add -A
git commit -m "Configura OpenXR e Meta Quest para Android"
```

---

### Task 4: Importar os modelos do Sebrae Lab [Gilberto]

**Files:**
- Create: `Assets/Sebraelab3D/` (27 FBX, 4 texturas, 1 `.blend1`)

**Interfaces:**
- Consumes: `C:\Users\Gilberto Jr\Downloads\SebraeLabUnityAssets.unitypackage`
- Produces: modelos disponíveis em `Assets/Sebraelab3D/`

- [ ] **Step 1: Importar o pacote**

**Assets › Import Package › Custom Package...** › escolher `Downloads\SebraeLabUnityAssets.unitypackage` › **Import** com tudo marcado.

- [ ] **Step 2: Conferir os materiais**

Abrir `Assets/Sebraelab3D`, clicar em `Sebrae Lab.fbx` e olhar a pré-visualização no Inspector. Se aparecer **rosa**, o material veio do pipeline antigo: **Edit › Rendering › Materials › Convert Selected Built-in Materials to URP** com os materiais selecionados.

*Por quê:* rosa na Unity significa shader incompatível. O projeto usa URP, e modelos exportados de outro lugar às vezes trazem materiais do pipeline padrão.

- [ ] **Step 3: Fechar a Unity e verificar [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
ls Assets/Sebraelab3D/*.fbx | wc -l
git status --short Assets/Sebraelab3D | head -3
```

Expected: `27` e arquivos novos listados.

- [ ] **Step 4: Remover o `.blend1` [Agente]**

`Sebrae Lab.blend1` é um backup automático do Blender, sem uso na Unity. Fica fora do repositório:

```bash
printf '\n# Backups do Blender\n*.blend1\n*.blend1.meta\n' >> .gitignore
```

- [ ] **Step 5: Commit [Agente]**

```bash
git add -A
git commit -m "Importa os modelos 3D do Sebrae Lab"
```

---

### Task 5: Estrutura de pastas, assemblies e testes [Agente]

**Files:**
- Create: `Assets/SebraeLab/{Scenes,Prefabs,Scripts/Interacoes,Dialogos,Materiais,Modelos,Audio,Tests/EditMode}/`
- Create: `Assets/SebraeLab/Scripts/SebraeLab.asmdef`
- Create: `Assets/SebraeLab/Tests/EditMode/SebraeLab.Tests.EditMode.asmdef`
- Create: `Assets/SebraeLab/Tests/EditMode/SanidadeTests.cs`

**Interfaces:**
- Consumes: Task 3 (pacotes XR presentes)
- Produces: assembly `SebraeLab` (referencia `Unity.XR.Interaction.Toolkit`, `Unity.TextMeshPro`) e assembly de testes `SebraeLab.Tests.EditMode`

- [ ] **Step 1: Criar as pastas**

```bash
mkdir -p "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2/Assets/SebraeLab"
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2/Assets/SebraeLab"
mkdir -p Scenes Prefabs Scripts/Interacoes Dialogos Materiais Modelos Audio Tests/EditMode
for d in Scenes Prefabs Dialogos Materiais Modelos Audio; do touch "$d/.gitkeep"; done
```

O git não versiona pasta vazia. O `.gitkeep` mantém a estrutura no repositório até cada pasta receber conteúdo.

- [ ] **Step 2: Assembly do código**

`Assets/SebraeLab/Scripts/SebraeLab.asmdef`:

```json
{
    "name": "SebraeLab",
    "rootNamespace": "SebraeLab",
    "references": [
        "Unity.XR.Interaction.Toolkit",
        "Unity.TextMeshPro"
    ],
    "includePlatforms": [],
    "excludePlatforms": [],
    "allowUnsafeCode": false,
    "overrideReferences": false,
    "precompiledReferences": [],
    "autoReferenced": true,
    "defineConstraints": [],
    "versionDefines": [],
    "noEngineReferences": false
}
```

- [ ] **Step 3: Assembly de testes**

`Assets/SebraeLab/Tests/EditMode/SebraeLab.Tests.EditMode.asmdef`:

```json
{
    "name": "SebraeLab.Tests.EditMode",
    "rootNamespace": "SebraeLab.Tests",
    "references": [
        "SebraeLab",
        "Unity.XR.Interaction.Toolkit",
        "UnityEngine.TestRunner",
        "UnityEditor.TestRunner"
    ],
    "includePlatforms": [
        "Editor"
    ],
    "excludePlatforms": [],
    "allowUnsafeCode": false,
    "overrideReferences": true,
    "precompiledReferences": [
        "nunit.framework.dll"
    ],
    "autoReferenced": false,
    "defineConstraints": [
        "UNITY_INCLUDE_TESTS"
    ],
    "versionDefines": [],
    "noEngineReferences": false
}
```

- [ ] **Step 4: Teste de sanidade**

`Assets/SebraeLab/Tests/EditMode/SanidadeTests.cs`:

```csharp
using NUnit.Framework;

namespace SebraeLab.Tests
{
    public class SanidadeTests
    {
        [Test]
        public void TestRunner_Executa()
        {
            Assert.AreEqual(2, 1 + 1);
        }
    }
}
```

- [ ] **Step 5: Rodar os testes**

Comando da seção "Rodar os testes". Expected: `result="Passed"`, `total="1"`, `failed="0"`.

Se falhar com erro de compilação citando assembly não encontrado, conferir os nomes em `Library/PackageCache/*/Runtime/*.asmdef` e corrigir as referências.

- [ ] **Step 6: Commit**

A Unity em modo batch gera os `.meta` dos arquivos novos. Eles entram no commit:

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A && git status --short | grep -c '\.meta$'
git commit -m "Cria estrutura de pastas e assemblies do projeto"
```

---

### Task 6: Montar a cena do Sebrae Lab [Gilberto]

**Files:**
- Create: `Assets/SebraeLab/Scenes/SebraeLab.unity`

**Interfaces:**
- Consumes: Tasks 3 e 4
- Produces: cena `SebraeLab` com o prédio, o jogador e locomoção. Hierarquia com os vazios `_Ambiente`, `_XR`, `_UI`, `_Audio`, `Interacoes`, `Missoes`.

- [ ] **Step 1: Criar a cena**

**File › New Scene** › **Basic (URP)** › **Create**. **File › Save As** › `Assets/SebraeLab/Scenes/SebraeLab.unity`. Apagar a `Main Camera` que veio na cena (o XR Origin traz a própria câmera).

- [ ] **Step 2: Organizar a hierarquia**

Botão direito na Hierarchy › **Create Empty**, seis vezes, com os nomes: `_Ambiente`, `_XR`, `_UI`, `_Audio`, `Interacoes`, `Missoes`. Zerar o Transform de todos. Dentro de `Interacoes`, criar os vazios `ItensDeInovacao`, `MuralDoConhecimento`, `NPC_Gerente`, `Boliche`, `ArcoEFlecha`. Mover a `Directional Light` para dentro de `_Ambiente`.

*Por quê:* vazios funcionam como pastas na cena. Em semanas, com dezenas de objetos, isso evita procurar coisas.

- [ ] **Step 3: Colocar o prédio**

Arrastar `Assets/Sebraelab3D/Sebrae Lab.fbx` para dentro de `_Ambiente`. Posição `0, 0, 0`.

Conferir a escala: selecionar uma porta ou mesa e comparar com o cubo padrão (**3D Object › Cube**, que mede 1 m). Uma porta tem cerca de 2,1 m de altura. Se o prédio estiver gigante ou minúsculo, ajustar o **Scale Factor** no Inspector do `.fbx` (aba Model), **Apply**. Apagar o cubo depois.

*Por quê:* em VR escala errada é óbvia e dá enjoo. FBX exportado do Blender às vezes chega 100 vezes maior ou menor.

Observação: ajustar o Scale Factor muda o `.meta` do FBX no pacote oficial. É configuração de importação, não edição do modelo, então é permitido.

- [ ] **Step 4: Dar colisão ao prédio**

Selecionar `Sebrae Lab` na Hierarchy. No Inspector do `.fbx` (aba Model), marcar **Generate Colliders** › **Apply**.

*Por quê:* sem collider, o jogador atravessa paredes e cai pelo chão. Gerar colliders na importação cria um Mesh Collider em cada peça do modelo.

- [ ] **Step 5: Colocar o jogador**

Arrastar para dentro de `_XR`:
- `Assets/Samples/XR Interaction Toolkit/3.0.11/Starter Assets/Prefabs/XR Origin (XR Rig).prefab`
- `Assets/Samples/XR Interaction Toolkit/3.0.11/XR Device Simulator/XR Device Simulator.prefab`

Posicionar o `XR Origin (XR Rig)` sobre o chão da recepção, com Y no nível do piso.

Se não houver `XR Interaction Manager` na cena: **GameObject › XR › Interaction Manager**, dentro de `_XR`.

Se não houver `EventSystem`: **GameObject › UI › Event System**. No Inspector, remover o `Standalone Input Module` (ou `Input System UI Input Module`) e adicionar **XR UI Input Module**. Mover para `_XR`.

*Por quê:* o XR Origin é o jogador (cabeça e mãos). O simulador finge ser o óculos no PC. O EventSystem com XR UI Input Module é o que permite apontar o raio do controle e clicar em botões, necessário para o mural da Task 9.

- [ ] **Step 6: Locomoção**

O XR Origin dos Starter Assets já traz movimento contínuo (analógico) e teleporte. O movimento contínuo funciona com os colliders do Step 4. Para o teleporte, adicionar ao objeto do piso o componente **Teleportation Area**.

Se o piso for parte de uma malha única com as paredes, criar **3D Object › Plane** do tamanho do piso, posicionado 1 cm acima, desmarcar **Mesh Renderer** e adicionar **Teleportation Area** nele.

*Por quê:* teleporte é a forma de locomoção que menos causa enjoo em quem não está acostumado com VR. Na apresentação, muita gente vai experimentar.

- [ ] **Step 7: Testar no simulador**

**Play**. Andar pela recepção, corredores e salas. Esperado: sem atravessar paredes, sem cair do chão, altura de olhar coerente com uma pessoa. Testar o teleporte.

- [ ] **Step 8: Salvar e commit [Agente, com a Unity fechada]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
rm -f Assets/SebraeLab/Scenes/.gitkeep
git add -A
git commit -m "Monta a cena do Sebrae Lab com jogador e locomoção"
```

---

### Task 7: Primeiro build no Quest [Gilberto]

**Files:**
- Modify: `ProjectSettings/EditorBuildSettings.asset`

**Interfaces:**
- Consumes: Task 6
- Produces: APK instalado e rodando no Quest

- [ ] **Step 1: Modo desenvolvedor no Quest**

1. Criar uma organização em developers.meta.com (gratuito, pede só um nome).
2. No celular, app **Meta Horizon** › **Dispositivos** › o óculos › **Configurações do headset** › **Modo de desenvolvedor** › ativar.
3. Reiniciar o óculos.

*Por quê:* sem modo desenvolvedor, o Quest só aceita apps da loja.

- [ ] **Step 2: Ligar no notebook**

Cabo USB-C do Quest no notebook. No óculos, aparece "Permitir depuração USB?": marcar **Sempre permitir deste computador** › **OK**.

Se o Windows não reconhecer o óculos, instalar o **Oculus ADB Driver** (developers.meta.com › Downloads).

- [ ] **Step 3: Incluir a cena no build**

**File › Build Profiles** › Android › **Scene List** › **Add Open Scenes** com a `SebraeLab` aberta. Desmarcar a `SampleScene` e a `DemoScene` se estiverem na lista.

- [ ] **Step 4: Build And Run**

Em **Run Device**, escolher o Quest (clicar em **Refresh** se não aparecer). **Build And Run**, salvar o APK em uma pasta `Builds/` na raiz do projeto (já ignorada pelo `.gitignore`).

O primeiro build demora bastante: a Unity compila shaders e código nativo para ARM64.

- [ ] **Step 5: Testar no óculos**

Esperado: o app abre direto no Sebrae Lab, a cabeça mexe a visão, os controles aparecem nas mãos, o analógico anda e o teleporte funciona.

Se a tela ficar preta ou o app fechar, enviar ao agente o log: **Window › Analysis › Android Logcat** (instalar o pacote se pedir) e copiar as linhas vermelhas.

- [ ] **Step 6: Apagar o APK e commit [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A
git diff --cached --name-only | grep -c '^Builds/'
git commit -m "Inclui a cena SebraeLab no build Android"
```

Expected: `0` (o `.gitignore` mantém o APK fora do git). O APK ocupa espaço num disco apertado e pode ser gerado de novo a qualquer momento. Apagá-lo fica a critério do Gilberto.

---

### Task 8: Interação 1, itens de inovação [Agente + Gilberto]

**Files:**
- Create: `Assets/SebraeLab/Scripts/Interacoes/ItemDeInovacao.cs`
- Create: `Assets/SebraeLab/Scripts/Interacoes/VirarParaCamera.cs`
- Test: `Assets/SebraeLab/Tests/EditMode/ItemDeInovacaoTests.cs`
- Create: `Assets/SebraeLab/Prefabs/ItemDeInovacao_Cartilha.prefab`, `ItemDeInovacao_Prototipo.prefab`, `ItemDeInovacao_Caneca.prefab`

**Interfaces:**
- Consumes: assembly `SebraeLab` (Task 5); `UnityEngine.XR.Interaction.Toolkit.Interactables.XRGrabInteractable`; `UnityEngine.XR.Interaction.Toolkit.SelectEnterEventArgs`
- Produces:
  - `SebraeLab.ItemDeInovacao : MonoBehaviour`
    - campos públicos: `string nome`, `string descricao`, `GameObject etiqueta`, `TMPro.TMP_Text textoDaEtiqueta`
    - `UnityEvent aoSerPegoPelaPrimeiraVez`
    - `bool JaFoiPego { get; }`
    - `void AoSerPego()`
    - `string TextoDaEtiqueta()`
  - `SebraeLab.VirarParaCamera : MonoBehaviour`

- [ ] **Step 1: Escrever os testes que falham [Agente]**

`Assets/SebraeLab/Tests/EditMode/ItemDeInovacaoTests.cs`:

```csharp
using NUnit.Framework;
using UnityEngine;

namespace SebraeLab.Tests
{
    public class ItemDeInovacaoTests
    {
        GameObject objeto;
        GameObject etiqueta;
        ItemDeInovacao item;

        [SetUp]
        public void Preparar()
        {
            objeto = new GameObject("Item");
            item = objeto.AddComponent<ItemDeInovacao>();
            etiqueta = new GameObject("Etiqueta");
            etiqueta.SetActive(false);
            item.etiqueta = etiqueta;
            item.nome = "Cartilha Sebrae";
            item.descricao = "Guia para abrir seu MEI";
        }

        [TearDown]
        public void Limpar()
        {
            Object.DestroyImmediate(objeto);
            Object.DestroyImmediate(etiqueta);
        }

        [Test]
        public void AoSerPego_PrimeiraVez_MostraEtiqueta()
        {
            item.AoSerPego();

            Assert.IsTrue(etiqueta.activeSelf);
            Assert.IsTrue(item.JaFoiPego);
        }

        [Test]
        public void AoSerPego_DuasVezes_DisparaEventoUmaVez()
        {
            var disparos = 0;
            item.aoSerPegoPelaPrimeiraVez.AddListener(() => disparos++);

            item.AoSerPego();
            item.AoSerPego();

            Assert.AreEqual(1, disparos);
        }

        [Test]
        public void TextoDaEtiqueta_ComDescricao_JuntaNomeEDescricao()
        {
            Assert.AreEqual("Cartilha Sebrae\n<size=70%>Guia para abrir seu MEI</size>", item.TextoDaEtiqueta());
        }

        [Test]
        public void TextoDaEtiqueta_SemDescricao_MostraSoONome()
        {
            item.descricao = "";

            Assert.AreEqual("Cartilha Sebrae", item.TextoDaEtiqueta());
        }

        [Test]
        public void AoSerPego_SemEtiquetaConfigurada_NaoQuebra()
        {
            item.etiqueta = null;

            Assert.DoesNotThrow(() => item.AoSerPego());
            Assert.IsTrue(item.JaFoiPego);
        }
    }
}
```

- [ ] **Step 2: Rodar e ver falhar [Agente]**

Comando da seção "Rodar os testes". Expected: falha de compilação, `ItemDeInovacao` não existe.

- [ ] **Step 3: Implementar [Agente]**

`Assets/SebraeLab/Scripts/Interacoes/ItemDeInovacao.cs`:

```csharp
using TMPro;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.XR.Interaction.Toolkit;
using UnityEngine.XR.Interaction.Toolkit.Interactables;

namespace SebraeLab
{
    /// <summary>
    /// Item que o jogador pode pegar. Na primeira pegada mostra uma etiqueta
    /// com nome e descrição e avisa quem estiver ouvindo (ex.: missões).
    /// </summary>
    [RequireComponent(typeof(XRGrabInteractable))]
    public class ItemDeInovacao : MonoBehaviour
    {
        public string nome = "Item";
        [TextArea(2, 4)] public string descricao = "";
        public GameObject etiqueta;
        public TMP_Text textoDaEtiqueta;

        public UnityEvent aoSerPegoPelaPrimeiraVez = new UnityEvent();

        public bool JaFoiPego { get; private set; }

        XRGrabInteractable pegavel;

        void Awake()
        {
            pegavel = GetComponent<XRGrabInteractable>();
            if (etiqueta != null) etiqueta.SetActive(false);
        }

        void OnEnable()
        {
            pegavel.selectEntered.AddListener(QuandoSelecionado);
        }

        void OnDisable()
        {
            pegavel.selectEntered.RemoveListener(QuandoSelecionado);
        }

        void QuandoSelecionado(SelectEnterEventArgs args) => AoSerPego();

        public void AoSerPego()
        {
            if (JaFoiPego) return;
            JaFoiPego = true;

            if (textoDaEtiqueta != null) textoDaEtiqueta.text = TextoDaEtiqueta();
            if (etiqueta != null) etiqueta.SetActive(true);

            aoSerPegoPelaPrimeiraVez.Invoke();
        }

        public string TextoDaEtiqueta()
        {
            return string.IsNullOrEmpty(descricao)
                ? nome
                : $"{nome}\n<size=70%>{descricao}</size>";
        }
    }
}
```

`Assets/SebraeLab/Scripts/Interacoes/VirarParaCamera.cs`:

```csharp
using UnityEngine;

namespace SebraeLab
{
    /// <summary>
    /// Mantém o objeto (etiqueta, placa) virado para o jogador.
    /// </summary>
    public class VirarParaCamera : MonoBehaviour
    {
        Transform cameraDoJogador;

        void LateUpdate()
        {
            if (cameraDoJogador == null)
            {
                if (Camera.main == null) return;
                cameraDoJogador = Camera.main.transform;
            }

            var direcao = transform.position - cameraDoJogador.position;
            if (direcao.sqrMagnitude < 0.0001f) return;
            transform.rotation = Quaternion.LookRotation(direcao, Vector3.up);
        }
    }
}
```

`direcao` aponta da câmera para a etiqueta. Com o +Z da etiqueta nessa direção, um texto TextMeshPro fica legível de frente para o jogador.

- [ ] **Step 4: Rodar e ver passar [Agente]**

Expected: `result="Passed"`, `total="6"` (5 novos e o de sanidade), `failed="0"`.

- [ ] **Step 5: Commit [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A
git commit -m "Adiciona interação de pegar itens de inovação"
```

- [ ] **Step 6: Montar o primeiro item [Gilberto]**

Na cena `SebraeLab`, dentro de `Interacoes/ItensDeInovacao`:

1. **Create Empty**, nome `Cartilha`, Scale `1, 1, 1`. Colocar sobre uma mesa da recepção.
2. Filho visual: botão direito em `Cartilha` › **3D Object › Cube**, nome `Modelo`. Posição local `0, 0, 0`, Scale `0.21, 0.01, 0.297` (folha A4 em metros). O cubo já vem com **Box Collider**.
3. Em `Cartilha` (o pai): **Add Component › ItemDeInovacao**. A Unity adiciona sozinha `XR Grab Interactable` e `Rigidbody`, que o script exige. O `XR Grab Interactable` encontra o collider do filho `Modelo` sozinho.
4. Filho etiqueta: botão direito em `Cartilha` › **3D Object › Text - TextMeshPro**, nome `Etiqueta`. Se a Unity pedir **Import TMP Essentials**, aceitar. Posição local `0, 0.15, 0`, Scale `1, 1, 1`, Rect Transform Width `0.4` e Height `0.15`, Font Size `0.5`, alinhamento centralizado. **Add Component › VirarParaCamera**.
5. No `ItemDeInovacao` da `Cartilha`: arrastar `Etiqueta` para os campos **Etiqueta** e **Texto Da Etiqueta**. Preencher **Nome** `Cartilha Sebrae` e **Descricao** `Guia para quem quer abrir o próprio negócio`.

*Por quê:* o `XR Grab Interactable` faz o objeto poder ser agarrado. O `Rigidbody` dá peso e faz cair ao soltar. O nosso script só cuida do que é específico do projeto: etiqueta e aviso.

*Por que pai vazio com dois filhos:* a escala do pai passa para os filhos. Se a etiqueta fosse filha do cubo achatado, herdaria a escala desigual e ficaria deformada ao girar para a câmera. Com o pai em escala 1, o formato fica no `Modelo` e a etiqueta fica livre. As duas acompanham o item quando ele é carregado.

- [ ] **Step 7: Testar no simulador [Gilberto]**

**Play**. Aproximar, agarrar a cartilha com o controle simulado, girar, soltar. Esperado: a etiqueta aparece na primeira pegada, fica virada para a câmera e continua visível depois. A cartilha cai na mesa ao soltar.

- [ ] **Step 8: Virar prefab e repetir [Gilberto]**

Arrastar `Cartilha` da Hierarchy para `Assets/SebraeLab/Prefabs/` e renomear o arquivo para `ItemDeInovacao_Cartilha`. Repetir os Steps 6 e 7 para mais dois itens, mesma estrutura (pai vazio em escala 1, filhos `Modelo` e `Etiqueta`), trocando só o `Modelo`:
- `ItemDeInovacao_Prototipo`: `Modelo` é **3D Object › Cylinder**, Scale `0.08, 0.06, 0.08`. Nome "Protótipo de produto", descrição "Ideia validada no laboratório de inovação".
- `ItemDeInovacao_Caneca`: `Modelo` é **3D Object › Capsule**, Scale `0.08, 0.05, 0.08`. Nome "Caneca Sebrae", descrição "Brinde clássico de evento".

Espalhar os três em mesas diferentes.

- [ ] **Step 9: Commit [Agente, com a Unity fechada]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
rm -f Assets/SebraeLab/Prefabs/.gitkeep
git add -A
git commit -m "Coloca três itens de inovação nas mesas do Sebrae Lab"
```

---

### Task 9: Interação 2, Mural do Conhecimento [Agente + Gilberto]

**Files:**
- Create: `Assets/SebraeLab/Scripts/Interacoes/PainelInformativo.cs`
- Test: `Assets/SebraeLab/Tests/EditMode/PainelInformativoTests.cs`
- Create: `Assets/SebraeLab/Prefabs/MuralDoConhecimento.prefab`

**Interfaces:**
- Consumes: assembly `SebraeLab`; `UnityEngine.XR.Interaction.Toolkit.Interactables.XRSimpleInteractable`; `UnityEngine.XR.Interaction.Toolkit.SelectEnterEventArgs`
- Produces:
  - `SebraeLab.PainelInformativo : MonoBehaviour`
    - campos públicos: `XRSimpleInteractable gatilho`, `GameObject painel`, `TMP_Text titulo`, `TMP_Text corpo`, `string textoTitulo`, `string textoCorpo`, `float distancia`
    - `UnityEvent aoAbrir`, `UnityEvent aoFechar`
    - `bool EstaAberto { get; }`
    - `void Abrir()`, `void Fechar()`
    - `static (Vector3 posicao, Quaternion rotacao) CalcularPose(Vector3 posicaoCamera, Vector3 frenteCamera, float distancia)`

`aoFechar` é o evento "mural lido" que a missão 3 vai escutar. O painel de boas-vindas da Parte 4 reaproveita este mesmo componente, sem `gatilho`.

- [ ] **Step 1: Escrever os testes que falham [Agente]**

`Assets/SebraeLab/Tests/EditMode/PainelInformativoTests.cs`:

```csharp
using NUnit.Framework;
using UnityEngine;

namespace SebraeLab.Tests
{
    public class PainelInformativoTests
    {
        GameObject objeto;
        GameObject painel;
        PainelInformativo mural;

        [SetUp]
        public void Preparar()
        {
            objeto = new GameObject("Mural");
            mural = objeto.AddComponent<PainelInformativo>();
            painel = new GameObject("Painel");
            painel.SetActive(false);
            mural.painel = painel;
        }

        [TearDown]
        public void Limpar()
        {
            Object.DestroyImmediate(objeto);
            Object.DestroyImmediate(painel);
        }

        [Test]
        public void Abrir_MostraPainelEDisparaEvento()
        {
            var aberturas = 0;
            mural.aoAbrir.AddListener(() => aberturas++);

            mural.Abrir();

            Assert.IsTrue(painel.activeSelf);
            Assert.IsTrue(mural.EstaAberto);
            Assert.AreEqual(1, aberturas);
        }

        [Test]
        public void Abrir_JaAberto_NaoDisparaDeNovo()
        {
            var aberturas = 0;
            mural.aoAbrir.AddListener(() => aberturas++);

            mural.Abrir();
            mural.Abrir();

            Assert.AreEqual(1, aberturas);
        }

        [Test]
        public void Fechar_EscondePainelEDisparaEvento()
        {
            var fechamentos = 0;
            mural.aoFechar.AddListener(() => fechamentos++);
            mural.Abrir();

            mural.Fechar();

            Assert.IsFalse(painel.activeSelf);
            Assert.IsFalse(mural.EstaAberto);
            Assert.AreEqual(1, fechamentos);
        }

        [Test]
        public void Fechar_SemEstarAberto_NaoDisparaEvento()
        {
            var fechamentos = 0;
            mural.aoFechar.AddListener(() => fechamentos++);

            mural.Fechar();

            Assert.AreEqual(0, fechamentos);
        }

        [Test]
        public void CalcularPose_CabecaInclinada_PoeNaAlturaDosOlhos()
        {
            var olhos = new Vector3(0f, 1.6f, 0f);
            var frente = new Vector3(0f, -0.5f, 1f).normalized;

            var (posicao, rotacao) = PainelInformativo.CalcularPose(olhos, frente, 1.2f);

            Assert.That(Vector3.Distance(posicao, new Vector3(0f, 1.6f, 1.2f)), Is.LessThan(0.001f));
            Assert.That(Vector3.Angle(rotacao * Vector3.forward, Vector3.forward), Is.LessThan(0.01f));
        }

        [Test]
        public void CalcularPose_OlhandoNaVertical_UsaFrentePadrao()
        {
            var olhos = new Vector3(2f, 1.6f, 3f);

            var (posicao, _) = PainelInformativo.CalcularPose(olhos, Vector3.down, 1f);

            Assert.That(Vector3.Distance(posicao, new Vector3(2f, 1.6f, 4f)), Is.LessThan(0.001f));
        }
    }
}
```

- [ ] **Step 2: Rodar e ver falhar [Agente]**

Expected: falha de compilação, `PainelInformativo` não existe.

- [ ] **Step 3: Implementar [Agente]**

`Assets/SebraeLab/Scripts/Interacoes/PainelInformativo.cs`:

```csharp
using TMPro;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.XR.Interaction.Toolkit;
using UnityEngine.XR.Interaction.Toolkit.Interactables;

namespace SebraeLab
{
    /// <summary>
    /// Painel de texto que abre diante do jogador e fecha no botão "Entendido".
    /// Pode ser acionado por um objeto (gatilho) ou chamado direto por código.
    /// </summary>
    public class PainelInformativo : MonoBehaviour
    {
        public XRSimpleInteractable gatilho;
        public GameObject painel;
        public TMP_Text titulo;
        public TMP_Text corpo;
        public string textoTitulo = "Mural do Conhecimento";
        [TextArea(4, 12)] public string textoCorpo = "";
        public float distancia = 1.2f;

        public UnityEvent aoAbrir = new UnityEvent();
        public UnityEvent aoFechar = new UnityEvent();

        public bool EstaAberto { get; private set; }

        void Awake()
        {
            if (painel != null) painel.SetActive(false);
        }

        void OnEnable()
        {
            if (gatilho != null) gatilho.selectEntered.AddListener(QuandoAcionado);
        }

        void OnDisable()
        {
            if (gatilho != null) gatilho.selectEntered.RemoveListener(QuandoAcionado);
        }

        void QuandoAcionado(SelectEnterEventArgs args) => Abrir();

        public void Abrir()
        {
            if (EstaAberto) return;
            EstaAberto = true;

            if (titulo != null) titulo.text = textoTitulo;
            if (corpo != null) corpo.text = textoCorpo;

            var cameraDoJogador = Camera.main;
            if (painel != null && cameraDoJogador != null)
            {
                var (posicao, rotacao) = CalcularPose(
                    cameraDoJogador.transform.position,
                    cameraDoJogador.transform.forward,
                    distancia);
                painel.transform.SetPositionAndRotation(posicao, rotacao);
            }

            if (painel != null) painel.SetActive(true);
            aoAbrir.Invoke();
        }

        public void Fechar()
        {
            if (!EstaAberto) return;
            EstaAberto = false;

            if (painel != null) painel.SetActive(false);
            aoFechar.Invoke();
        }

        /// <summary>
        /// Posição à frente do jogador, na altura dos olhos, ignorando para onde
        /// ele inclina a cabeça. O +Z do painel aponta para longe do jogador,
        /// que é a orientação em que um Canvas fica legível.
        /// </summary>
        public static (Vector3 posicao, Quaternion rotacao) CalcularPose(
            Vector3 posicaoCamera, Vector3 frenteCamera, float distancia)
        {
            var frente = Vector3.ProjectOnPlane(frenteCamera, Vector3.up);
            if (frente.sqrMagnitude < 0.000001f) frente = Vector3.forward;
            frente.Normalize();

            return (posicaoCamera + frente * distancia, Quaternion.LookRotation(frente, Vector3.up));
        }
    }
}
```

- [ ] **Step 4: Rodar e ver passar [Agente]**

Expected: `result="Passed"`, `total="12"`, `failed="0"`.

- [ ] **Step 5: Commit [Agente]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A
git commit -m "Adiciona painel informativo do Mural do Conhecimento"
```

- [ ] **Step 6: Montar o mural [Gilberto]**

Dentro de `Interacoes/MuralDoConhecimento`:

1. Arrastar `Assets/Sebraelab3D/SM_Quadro.fbx` e posicionar numa parede visível do percurso. Adicionar **Box Collider** (o raio do controle só detecta o que tem collider) e **XR Simple Interactable**.
2. No objeto `MuralDoConhecimento`: **Add Component › PainelInformativo**. Arrastar o quadro para o campo **Gatilho**.
3. Painel: botão direito em `MuralDoConhecimento` › **UI › Canvas**, nome `Painel`.
   - **Render Mode**: World Space. **Event Camera**: a `Main Camera` dentro do `XR Origin`.
   - **Rect Transform**: Width `800`, Height `600`, Scale `0.001, 0.001, 0.001` (vira 80 × 60 cm).
   - Remover **Graphic Raycaster** e adicionar **Tracked Device Graphic Raycaster**.
   - Filhos: **UI › Image** de fundo (esticada, cor escura com leve transparência), **UI › Text - TextMeshPro** `Titulo` no topo, **UI › Text - TextMeshPro** `Corpo` no meio com quebra de linha, **UI › Button - TextMeshPro** `Entendido` embaixo.
4. No `PainelInformativo`: arrastar `Painel`, `Titulo` e `Corpo` para os campos. Escrever o **Texto Corpo** com 3 a 5 frases sobre o Sebrae Lab (o que é, para quem, o que oferece).
5. No botão `Entendido`, em **On Click ()**: **+**, arrastar `MuralDoConhecimento`, escolher **PainelInformativo › Fechar**.

*Por quê:* `XR Simple Interactable` é o "botão" mais simples do XR Toolkit: não agarra, só avisa que foi acionado. O Canvas em World Space é UI que existe no espaço 3D, como uma placa. O `Tracked Device Graphic Raycaster` permite que o raio do controle clique nos botões; sem ele, o raio atravessa o painel.

- [ ] **Step 7: Testar no simulador [Gilberto]**

**Play**. Apontar o raio para o quadro e apertar o gatilho. Esperado: o painel abre a cerca de 1,2 m, na altura dos olhos e de frente. Ler o texto. Apontar para **Entendido** e apertar: o painel some. Acionar o quadro de novo: o painel reabre na nova posição do jogador.

- [ ] **Step 8: Prefab e commit [Gilberto + Agente]**

Arrastar `MuralDoConhecimento` para `Assets/SebraeLab/Prefabs/`. Salvar a cena e fechar a Unity.

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git add -A
git commit -m "Coloca o Mural do Conhecimento na cena"
```

---

### Task 10: Validação da semana no Quest [Gilberto + Agente]

**Files:** nenhum arquivo novo

**Interfaces:**
- Consumes: Tasks 6 a 9
- Produces: semana 1 validada no óculos e enviada ao GitHub

- [ ] **Step 1: Rodar a suíte de testes [Agente]**

Expected: `total="12"`, `failed="0"`.

- [ ] **Step 2: Build And Run no Quest [Gilberto]**

Mesmo processo da Task 7, Step 4.

- [ ] **Step 3: Roteiro de teste no óculos [Gilberto]**

1. App abre na recepção.
2. Andar e teleportar até as mesas.
3. Pegar os três itens; cada etiqueta aparece na primeira pegada e fica legível.
4. Soltar um item: ele cai e não atravessa a mesa.
5. Ir até o mural, acionar, ler, clicar em **Entendido**.
6. Atravessar paredes não deve ser possível.

Anotar o que falhar e passar ao agente.

- [ ] **Step 4: Push [Agente, com autorização do Gilberto]**

```bash
cd "/c/Users/Gilberto Jr/Downloads/Claude/Projetos/te2"
git log --oneline origin/main..HEAD
git push origin main
```

O próximo plano (semana 2: NPC e boliche) parte deste estado.

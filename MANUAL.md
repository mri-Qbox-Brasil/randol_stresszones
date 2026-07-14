# randol_stresszones — Manual

Zonas poligonais no mapa onde o jogador reduz estresse ao executar animações ou scenarios de relaxamento.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Como funciona](#como-funciona)
5. [Integrações](#integrações)
6. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | `lib.require`, `lib.zones.poly`, `lib.notify`, `cache.ped` |
| `qb-core` | Sim | `exports['qb-core']:GetCoreObject()` e leitura de `PlayerData.metadata.stress` |
| `qb-hud` (ou outro recurso que ouça `hud:server:RelieveStress`) | Sim | É quem efetivamente abaixa o estresse; sem ele o recurso não tem efeito |

---

## Instalação

1. Copie a pasta `randol_stresszones` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure randol_stresszones
   ```
3. Confirme que existe no servidor um recurso que trate o evento `hud:server:RelieveStress` (o `qb-hud` padrão trata).

Não há SQL nem itens de inventário.

---

## Configuração

Todo o config está em `config.lua`, que retorna uma tabela lida via `lib.require('config')`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `EnableBlips` | bool | Não | Cria um blip no mapa para cada zona (sprite 311, cor 23, nome "Stress Zone") |
| `Timer` | number (ms) | Sim | Intervalo entre cada tentativa de redução de estresse enquanto o jogador está dentro da zona |
| `Zones` | array | Sim | Lista de zonas. Cada entrada tem `blip` e `points` |
| `Zones[i].blip` | `vec3` | Sim | Coordenada onde o blip é desenhado |
| `Zones[i].points` | array de `vec3` | Sim | Pontos do polígono da zona (a espessura é fixa em 20) |
| `Animations` | array de `{dict, anim}` | Sim | Animações aceitas. Se o jogador estiver tocando uma delas dentro da zona, o estresse cai |
| `Scenarios` | array de string | Sim | Scenarios aceitos (ex.: `WORLD_HUMAN_YOGA`). Mesma função das animações |

Exemplo de zona:

```lua
{
    blip = vec3(-1251.14, -1501.67, 4.6),
    points = {
        vec3(-1233.73, -1513.49, 4.32),
        vec3(-1236.77, -1516.18, 4.32),
        vec3(-1276.81, -1507.12, 4.32),
    }
}
```

---

## Como funciona

1. As zonas são criadas quando o jogador carrega (`QBCore:Client:OnPlayerLoaded`) ou quando o recurso reinicia com o jogador já logado.
2. Ao entrar em uma zona, o jogador recebe uma notificação avisando para usar emotes de sentar/yoga.
3. A cada `Config.Timer` milissegundos, o recurso verifica se o ped está tocando alguma animação de `Config.Animations` **ou** usando algum scenario de `Config.Scenarios`. Se sim e o estresse for maior que zero, dispara `hud:server:RelieveStress` com um valor aleatório de 1 a 2.
4. Ao sair da zona, o loop para. As zonas são removidas no `QBCore:Client:OnPlayerUnload` e no stop do recurso.

O recurso **não** toca a animação por conta própria — o jogador precisa acioná-la por outro meio (menu de emotes, por exemplo). A notificação padrão sugere `sit3`, `sit4` e `yoga`.

---

## Integrações

### qb-hud

A redução de estresse é feita pelo evento de servidor `hud:server:RelieveStress`, disparado com um inteiro aleatório (1 ou 2):

```lua
TriggerServerEvent('hud:server:RelieveStress', math.random(2))
```

Qualquer recurso de HUD que registre esse evento funciona no lugar do `qb-hud`.

### Emotes

Para adicionar novas animações válidas, inclua o par `{dict, anim}` em `Config.Animations` (verificado com `IsEntityPlayingAnim`) ou o nome do scenario em `Config.Scenarios` (verificado com `IsPedUsingScenario`).

---

## Estrutura de arquivos

```
randol_stresszones/
├── cl_stress.lua      — criação/remoção das zonas, loop de checagem de animação e disparo do evento de estresse
├── config.lua         — zonas, blips, timer, animações e scenarios aceitos
├── fxmanifest.lua
└── README.md
```

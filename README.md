# Banner no perfil do TikTok

**Método criado 100% por @ripxeonno** — por favor dar créditos caso use ou repasse o tutorial.

O TikTok já tem fluxo de **plano de fundo do perfil** (“Adicionar plano de fundo”), mas o app **não mostra o botão** pra maioria das contas: várias checagens no cliente decidem se você entrou na lista interna, se o modo “immersive” vale, se o aparelho não é tratado como “fraco”, etc. O truque é **hookar o TikTok na hora que ele abre** e **forçar essas checagens como se a feature estivesse liberada** — sem depender do servidor te colocar na allow list só pra UI aparecer.

Funciona em cima do **TikTok 47.1.3** (`com.zhiliaoapp.musically`). Em outras versões os nomes ofuscados mudam e os hooks não acham a classe.

---

## O que o app verifica (e o que a gente contorna)

| Onde (47.1.3) | Método | Comportamento normal | O que fazemos |
|---------------|--------|----------------------|---------------|
| Classe gate `X.0OSG` | `LIZJ(boolean)` | Decide se a feature de banner/perfil “master” está ativa | Interceptar e **sempre retornar `true`** |
| Mesma classe `X.0OSG` | `LIZIZ()` | Gate do modo immersive ligado ao banner | Interceptar e **sempre retornar `true`** |
| `com.ss.android.ugc.profile.platform.business.background.ProfileBackgroundComponent` | `Ds()` | Trata aparelho como “low device” e esconde/limita background | Interceptar e **sempre retornar `false`** (não é aparelho fraco) |

Com isso a UI de **adicionar/trocar banner** volta a aparecer no perfil como se a conta fosse elegível.

*Créditos: @ripxeonno*

---

## Como os hooks entram no TikTok

1. **Escopo só no pacote do TikTok** — nada roda em outros apps.
2. Quando o TikTok sobe, engancha **`Application.attach`** e **`Application.onCreate`** para rodar a instalação **uma vez**, com o `ClassLoader` certo do app (importante se o loader for trocado depois do bootstrap).
3. Carrega as classes acima pelo nome e aplica **intercept** nos três métodos (dois retornos `true`, um `false` no gate de low-end).
4. **Força parada** no TikTok e abre de novo pra garantir que os hooks pegaram desde o início.

Sem root dá o mesmo efeito **embutindo esse código no APK do TikTok** (patch que injeta módulo no processo), em vez de carregar um módulo separado com root — mesma lógica de hooks, outro jeito de entregar.

---

## Depois que liberou a UI

1. Perfil → **Adicionar plano de fundo** (ou equivalente) e escolhe a imagem.
2. O banner **sobe pros servidores do TikTok**; outras contas **veem** no teu perfil.
3. Bug/comportamento do app: **no teu próprio perfil muitas vezes não renderiza o banner de volta** pra você. Pra saber se ficou certo, olha com **segunda conta** ou pede print de alguém.

---

## Logs (exemplo do que aparece quando dá certo)

Quando você aplica o método, reinicia o TikTok e abre o log do módulo / do patch, algo **nesse estilo** (valores mudam conforme aparelho e build):

```
ProfileBanner loader v1.0.4, Windows 10.0.26200, amd64
Target app prepared for hook refresh
Using package: com.zhiliaoapp.musically (TikTok 47.1.3) build 2025030412
Scope locked to TikTok only — other apps skipped
Resolving ClassLoader from Application.attach...
ClassLoader OK (PathClassLoader, dex count 8)
Prefetching gate classes...
Checking obfuscation map for 47.1.3...
Gate class X.0OSG found
ProfileBackgroundComponent found
Registering intercept tpb-master-gate on X.0OSG.LIZJ
Registering intercept tpb-immersive-gate on X.0OSG.LIZIZ
Registering intercept tpb-low-device-gate on ProfileBackgroundComponent.Ds
Enable profile banner setting: true
Installing hooks: 20% Application.attach
Installing hooks: 40% Application.onCreate
Installing hooks: 60% master gate LIZJ forced true
Installing hooks: 75% immersive gate LIZIZ forced true
Installing hooks: 90% low-device gate Ds forced false
Completed profile banner force hook installation [TikTok 47.1.3]
Connecting to TikTok process...
Cold start: force-stop acknowledged
Launching com.zhiliaoapp.musically...
Profile UI: 30% loading tabs
Profile UI: 60% background component init
Profile UI: 100% Add background entry visible
Upload pipeline: staging image...
Upload pipeline: 100% server accepted cover
Done.
```

Se alguma linha vier `master gate class unavailable` ou `LIZJ unavailable`, a versão do TikTok **não é 47.1.3** (ou o APK não é o mesmo build) — corrige a versão e repete.

---

## Resumo (cola no chat)

> Método **100% @ripxeonno** — dá crédito se usar.  
> Banner no perfil TikTok **47.1.3**: hook na abertura, forço `X.0OSG.LIZJ` + `LIZIZ` → true e `ProfileBackgroundComponent.Ds` → false; botão de fundo aparece sem estar “oficial” na conta. Reinicia o app, seta o banner. Confere com **outra conta** (no teu perfil o TikTok muitas vezes não mostra o banner de volta).

---

## Créditos e licença

- **Autoria do método:** @ripxeonno — **por favor dar créditos** se você usar, copiar o tutorial ou redistribuir o esquema.
- Código derivado / projeto: GPL-3.0 — ver `LICENSE` (isso não substitui o crédito acima).

# ПОЛЯНА — Implementation Plan (Demo)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Построить играбельное демо ПОЛЯНЫ — autochess с 3 архетипами юнитов, системой покупки и боем против ботов на Unity.

**Architecture:** Форк опенсорс autochess Unity-проекта как основа механики. Поверх — замена визуала, добавление новых архетипов и классов ПОЛЯНЫ. Управление Unity Editor через MCP прямо из терминала.

**Tech Stack:** Unity 2022.3 LTS, C#, URP, CoderGamester/mcp-unity, Meshy.ai (ассеты), Mixamo (анимации)

---

## Предварительные требования

Перед стартом задач выполнить вручную:
- [ ] Установить Unity Hub → скачать Unity 2022.3 LTS
- [ ] Создать аккаунт на [Meshy.ai](https://meshy.ai) (бесплатный тариф)
- [ ] Создать аккаунт на [Mixamo](https://mixamo.com) (бесплатно, нужен Adobe ID)
- [ ] Установить [CoderGamester/mcp-unity](https://github.com/CoderGamester/mcp-unity) по инструкции в README

---

## Task 0: Найти и форкнуть опенсорс основу

**Цель:** найти Unity autochess репо с открытым кодом, который реализует базовую механику (сетка + авто-бой). Использовать как фундамент вместо написания с нуля.

**Файлы:** — (исследование, не код)

- [ ] **Шаг 1: Поиск репо**

Агент выполняет поиск на GitHub по запросам:
```
site:github.com unity autochess "auto chess" C# open source
site:github.com unity "auto battle" grid turn-based
```
Критерии выбора: Unity C#, есть сетка (hex или квадрат), есть базовый авто-бой, лицензия MIT/Apache, последний коммит не старше 2022 года.

- [ ] **Шаг 2: Форкнуть выбранный репо в `antioz/polana`**

```bash
# Клонировать форк локально
git clone https://github.com/antioz/polana.git
cd polana
```

- [ ] **Шаг 3: Открыть проект в Unity**

Unity Hub → Open → выбрать папку проекта. Убедиться что проект запускается без ошибок.

- [ ] **Шаг 4: Коммит**
```bash
git add .
git commit -m "chore: initial unity project from open source base"
git push origin main
```

---

## Task 1: Подключить Unity MCP

**Цель:** Claude Code должен управлять Unity Editor через MCP — создавать объекты, писать скрипты, настраивать сцены.

**Файлы:**
- Modify: `Packages/manifest.json` (добавить mcp-unity пакет)
- Create: `.mcp/settings.json` (конфиг MCP для Claude Code)

- [ ] **Шаг 1: Установить пакет mcp-unity в Unity**

В Unity: Window → Package Manager → Add package from git URL:
```
https://github.com/CoderGamester/mcp-unity.git
```

- [ ] **Шаг 2: Запустить MCP сервер в Unity**

В Unity: Tools → MCP Unity → Start Server  
Убедиться что в консоли появилось: `MCP Server started on port 8090`

- [ ] **Шаг 3: Добавить MCP в Claude Code**

В терминале Cursor:
```bash
# Добавить в .claude/settings.json
{
  "mcpServers": {
    "unity": {
      "command": "node",
      "args": ["path/to/mcp-unity/server/build/index.js"],
      "env": { "UNITY_PORT": "8090" }
    }
  }
}
```

- [ ] **Шаг 4: Проверить соединение**

В Claude Code написать: «создай пустой GameObject с именем TestObject в текущей сцене Unity». Убедиться что объект появился в Hierarchy.

- [ ] **Шаг 5: Коммит**
```bash
git add Packages/ .claude/
git commit -m "feat: connect Unity MCP for agent-driven development"
git push origin main
```

---

## Task 2: Базовая сцена — Поляна

**Цель:** создать сцену с полем боя, сеткой, камерой и фоновыми персонажами (placeholder).

**Файлы:**
- Create: `Assets/Scenes/GameScene.unity`
- Create: `Assets/Scripts/Grid/GridManager.cs`
- Create: `Assets/Scripts/Grid/GridCell.cs`

- [ ] **Шаг 1: Создать сцену через MCP**

Агент выполняет через MCP:
```
Создай новую сцену GameScene. Добавь Plane (масштаб 10x10, зелёный материал — имитация травы). Настрой камеру: позиция (0, 15, -8), поворот (60, 0, 0), проекция Perspective.
```

- [ ] **Шаг 2: Написать GridManager.cs**

```csharp
// Assets/Scripts/Grid/GridManager.cs
using UnityEngine;

public class GridManager : MonoBehaviour
{
    public int rows = 4;
    public int cols = 6;
    public float cellSize = 1.5f;
    public GridCell cellPrefab;

    private GridCell[,] grid;

    void Start()
    {
        grid = new GridCell[rows, cols];
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
            {
                Vector3 pos = new Vector3(c * cellSize, 0, r * cellSize);
                var cell = Instantiate(cellPrefab, pos, Quaternion.identity, transform);
                cell.row = r;
                cell.col = c;
                grid[r, c] = cell;
            }
    }

    public GridCell GetCell(int row, int col) => grid[row, col];
}
```

- [ ] **Шаг 3: Написать GridCell.cs**

```csharp
// Assets/Scripts/Grid/GridCell.cs
using UnityEngine;

public class GridCell : MonoBehaviour
{
    public int row;
    public int col;
    public UnitBase occupant;

    public bool IsEmpty => occupant == null;
}
```

- [ ] **Шаг 4: Запустить сцену в Unity, убедиться что сетка отображается**

Play Mode → сетка 4x6 должна быть видна сверху.

- [ ] **Шаг 5: Добавить placeholder фоновых персонажей**

Через MCP: создать Capsule объекты по краям поля с именами `NPC_Grandpa`, `NPC_Mom`, `NPC_Cyclist`, `NPC_Camera_Left`, `NPC_Camera_Right`, `NPC_Coach_Left`, `NPC_Coach_Right`.

- [ ] **Шаг 6: Коммит**
```bash
git add Assets/Scenes/ Assets/Scripts/Grid/
git commit -m "feat: base scene with grid and NPC placeholders"
git push origin main
```

---

## Task 3: Базовый юнит — UnitBase

**Цель:** создать базовый класс юнита с HP, уроном, архетипом. Все юниты наследуются от него.

**Файлы:**
- Create: `Assets/Scripts/Units/UnitBase.cs`
- Create: `Assets/Scripts/Units/UnitArchetype.cs`

- [ ] **Шаг 1: Написать UnitArchetype.cs**

```csharp
// Assets/Scripts/Units/UnitArchetype.cs
public enum UnitArchetype
{
    Sportik,    // Подписной спортик
    Otmorozok,  // Отморозок
    Vyezdnoy    // Выездной
}
```

- [ ] **Шаг 2: Написать UnitBase.cs**

```csharp
// Assets/Scripts/Units/UnitBase.cs
using UnityEngine;

public class UnitBase : MonoBehaviour
{
    [Header("Stats")]
    public string unitName;
    public UnitArchetype archetype;
    public int teamId; // 0 = игрок, 1 = бот

    public float maxHp = 100f;
    public float currentHp;
    public float damage = 15f;
    public float attackRange = 1.5f;
    public float attackSpeed = 1f; // атак в секунду
    public float moveSpeed = 2f;

    [Header("State")]
    public GridCell currentCell;
    public UnitBase currentTarget;
    public bool isAlive = true;

    void Awake() => currentHp = maxHp;

    public virtual void TakeDamage(float amount)
    {
        currentHp -= amount;
        if (currentHp <= 0) Die();
    }

    public virtual void Attack(UnitBase target)
    {
        target.TakeDamage(damage);
    }

    protected virtual void Die()
    {
        isAlive = false;
        currentCell.occupant = null;
        Destroy(gameObject, 0.5f);
    }

    public float DistanceTo(UnitBase other)
    {
        return Vector3.Distance(transform.position, other.transform.position);
    }
}
```

- [ ] **Шаг 3: Запустить Unity, убедиться что скрипты компилируются без ошибок**

В Console не должно быть красных ошибок.

- [ ] **Шаг 4: Коммит**
```bash
git add Assets/Scripts/Units/
git commit -m "feat: UnitBase and UnitArchetype foundation"
git push origin main
```

---

## Task 4: AI боя — автоматическая атака

**Цель:** юниты автоматически находят ближайшего врага и атакуют. Это ядро autochess механики.

**Файлы:**
- Create: `Assets/Scripts/Combat/CombatAI.cs`
- Create: `Assets/Scripts/Combat/BattleManager.cs`

- [ ] **Шаг 1: Написать CombatAI.cs**

```csharp
// Assets/Scripts/Combat/CombatAI.cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class CombatAI : MonoBehaviour
{
    private UnitBase unit;
    private float attackTimer;

    void Awake() => unit = GetComponent<UnitBase>();

    void Update()
    {
        if (!unit.isAlive) return;

        unit.currentTarget = FindNearestEnemy();
        if (unit.currentTarget == null) return;

        float dist = unit.DistanceTo(unit.currentTarget);
        if (dist <= unit.attackRange)
        {
            attackTimer += Time.deltaTime;
            if (attackTimer >= 1f / unit.attackSpeed)
            {
                attackTimer = 0;
                PerformAttack();
            }
        }
        else
        {
            MoveToward(unit.currentTarget.transform.position);
        }
    }

    protected virtual void PerformAttack()
    {
        unit.Attack(unit.currentTarget);
    }

    private UnitBase FindNearestEnemy()
    {
        var allUnits = FindObjectsOfType<UnitBase>();
        UnitBase nearest = null;
        float minDist = float.MaxValue;

        foreach (var u in allUnits)
        {
            if (!u.isAlive || u.teamId == unit.teamId) continue;
            float d = unit.DistanceTo(u);
            if (d < minDist) { minDist = d; nearest = u; }
        }
        return nearest;
    }

    private void MoveToward(Vector3 target)
    {
        transform.position = Vector3.MoveTowards(
            transform.position, target, unit.moveSpeed * Time.deltaTime);
    }
}
```

- [ ] **Шаг 2: Написать BattleManager.cs**

```csharp
// Assets/Scripts/Combat/BattleManager.cs
using UnityEngine;
using System.Linq;

public class BattleManager : MonoBehaviour
{
    public static BattleManager Instance;

    public bool battleActive = false;

    void Awake() => Instance = this;

    public void StartBattle()
    {
        battleActive = true;
        var allUnits = FindObjectsOfType<CombatAI>();
        foreach (var ai in allUnits)
            ai.enabled = true;
    }

    void Update()
    {
        if (!battleActive) return;

        var alive = FindObjectsOfType<UnitBase>().Where(u => u.isAlive).ToList();
        bool team0 = alive.Any(u => u.teamId == 0);
        bool team1 = alive.Any(u => u.teamId == 1);

        if (!team0 || !team1)
        {
            battleActive = false;
            int winner = team0 ? 0 : 1;
            Debug.Log($"Battle ended. Team {winner} wins.");
        }
    }
}
```

- [ ] **Шаг 3: Проверить — поставить 2 юнита вручную, запустить бой**

В Unity: создать 2 GameObject с UnitBase + CombatAI, teamId 0 и 1. Нажать StartBattle(). Один должен победить.

- [ ] **Шаг 4: Коммит**
```bash
git add Assets/Scripts/Combat/
git commit -m "feat: combat AI with auto-attack and battle manager"
git push origin main
```

---

## Task 5: Архетипы — разное поведение в бою

**Цель:** реализовать уникальные механики каждого архетипа.

**Файлы:**
- Create: `Assets/Scripts/Units/SportikAI.cs`
- Create: `Assets/Scripts/Units/OtmorozokAI.cs`
- Create: `Assets/Scripts/Units/VyezdnoyAI.cs`

- [ ] **Шаг 1: SportikAI.cs — оглушение при атаке**

```csharp
// Assets/Scripts/Units/SportikAI.cs
using UnityEngine;

public class SportikAI : CombatAI
{
    public float stunChance = 0.25f; // 25% шанс оглушить
    public float stunDuration = 1.5f;

    protected override void PerformAttack()
    {
        base.PerformAttack();
        if (Random.value < stunChance)
            StartCoroutine(StunTarget(unit.currentTarget));
    }

    System.Collections.IEnumerator StunTarget(UnitBase target)
    {
        var ai = target.GetComponent<CombatAI>();
        if (ai == null) yield break;
        ai.enabled = false;
        yield return new WaitForSeconds(stunDuration);
        if (target.isAlive) ai.enabled = true;
    }
}
```

- [ ] **Шаг 2: OtmorozokAI.cs — AoE удар, шанс бить своих**

```csharp
// Assets/Scripts/Units/OtmorozokAI.cs
using UnityEngine;

public class OtmorozokAI : CombatAI
{
    public float aoeRadius = 2f;
    public float friendlyFireChance = 0.15f; // 15% бить союзника

    protected override void PerformAttack()
    {
        // AoE: бьём всех в радиусе
        var allUnits = FindObjectsOfType<UnitBase>();
        foreach (var u in allUnits)
        {
            if (!u.isAlive) continue;
            bool isFriendly = u.teamId == unit.teamId;
            if (isFriendly && Random.value > friendlyFireChance) continue;
            if (unit.DistanceTo(u) <= aoeRadius)
                unit.Attack(u);
        }
    }
}
```

- [ ] **Шаг 3: VyezdnoyAI.cs — аура на союзников**

```csharp
// Assets/Scripts/Units/VyezdnoyAI.cs
using UnityEngine;

public class VyezdnoyAI : CombatAI
{
    public float auraRadius = 3f;
    public float damageBonus = 0.15f; // +15% урон союзникам
    private bool auraApplied = false;

    void Start()
    {
        ApplyAura();
    }

    void ApplyAura()
    {
        if (auraApplied) return;
        var allUnits = FindObjectsOfType<UnitBase>();
        foreach (var u in allUnits)
        {
            if (u == unit || u.teamId != unit.teamId) continue;
            if (unit.DistanceTo(u) <= auraRadius)
                u.damage *= (1 + damageBonus);
        }
        auraApplied = true;
    }
}
```

- [ ] **Шаг 4: Проверить каждый архетип в бою**

Расставить по 2 юнита каждого архетипа, запустить бой. Убедиться что Отморозок иногда бьёт своих, Спортик оглушает, Выездной даёт бонус.

- [ ] **Шаг 5: Коммит**
```bash
git add Assets/Scripts/Units/
git commit -m "feat: archetype-specific combat behaviors"
git push origin main
```

---

## Task 6: Магазин и экономика

**Цель:** игрок покупает юнитов за золото между раундами.

**Файлы:**
- Create: `Assets/Scripts/Shop/ShopManager.cs`
- Create: `Assets/Scripts/Shop/ShopSlot.cs`

- [ ] **Шаг 1: ShopManager.cs**

```csharp
// Assets/Scripts/Shop/ShopManager.cs
using UnityEngine;
using System.Collections.Generic;

public class ShopManager : MonoBehaviour
{
    public static ShopManager Instance;
    public int playerGold = 10;
    public int shopSize = 5;
    public List<UnitBase> unitPool; // заполнить в Inspector

    private List<UnitBase> currentOffer = new();

    void Awake() => Instance = this;

    public void RefreshShop()
    {
        currentOffer.Clear();
        for (int i = 0; i < shopSize; i++)
        {
            int idx = Random.Range(0, unitPool.Count);
            currentOffer.Add(unitPool[idx]);
        }
    }

    public bool TryBuy(int slotIndex, GridCell targetCell)
    {
        if (slotIndex >= currentOffer.Count) return false;
        var unitPrefab = currentOffer[slotIndex];
        int cost = GetCost(unitPrefab);
        if (playerGold < cost) return false;

        playerGold -= cost;
        var unit = Instantiate(unitPrefab, targetCell.transform.position, Quaternion.identity);
        unit.teamId = 0;
        unit.currentCell = targetCell;
        targetCell.occupant = unit;
        currentOffer[slotIndex] = null;
        return true;
    }

    private int GetCost(UnitBase unit)
    {
        return unit.archetype switch
        {
            UnitArchetype.Sportik => 3,
            UnitArchetype.Otmorozok => 4,
            UnitArchetype.Vyezdnoy => 2,
            _ => 3
        };
    }
}
```

- [ ] **Шаг 2: Проверить — RefreshShop() заполняет слоты, TryBuy() снимает золото**

Через Inspector вызвать RefreshShop() вручную (кнопка в Inspector или Debug.Log).

- [ ] **Шаг 3: Коммит**
```bash
git add Assets/Scripts/Shop/
git commit -m "feat: shop system with gold economy"
git push origin main
```

---

## Task 7: Простой UI

**Цель:** отобразить золото игрока, HP, кнопку «Начать бой».

**Файлы:**
- Create: `Assets/Scripts/UI/GameUI.cs`
- Create: `Assets/UI/GameCanvas.prefab` (через Unity Editor)

- [ ] **Шаг 1: Создать Canvas через MCP**

```
Создай UI Canvas в сцене GameScene. Добавь:
- Text (TMP) "GoldText" в верхнем левом углу — отображает золото
- Text (TMP) "RoundText" в верхнем центре — номер раунда
- Button "StartBattleButton" внизу по центру с текстом "НАЧАТЬ БОЙ"
```

- [ ] **Шаг 2: GameUI.cs**

```csharp
// Assets/Scripts/UI/GameUI.cs
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class GameUI : MonoBehaviour
{
    public TMP_Text goldText;
    public TMP_Text roundText;
    public Button startBattleButton;

    private int round = 1;

    void Start()
    {
        startBattleButton.onClick.AddListener(OnStartBattle);
        UpdateUI();
    }

    void Update() => UpdateUI();

    void UpdateUI()
    {
        goldText.text = $"Золото: {ShopManager.Instance?.playerGold ?? 0}";
        roundText.text = $"Раунд {round}";
    }

    void OnStartBattle()
    {
        startBattleButton.interactable = false;
        BattleManager.Instance.StartBattle();
        round++;
    }
}
```

- [ ] **Шаг 3: Запустить, проверить UI**

Золото отображается, кнопка запускает бой, раунд увеличивается.

- [ ] **Шаг 4: Коммит**
```bash
git add Assets/Scripts/UI/ Assets/UI/
git commit -m "feat: basic game UI with gold display and start button"
git push origin main
```

---

## Task 8: Простой бот-противник

**Цель:** бот автоматически расставляет случайных юнитов на свою половину поля.

**Файлы:**
- Create: `Assets/Scripts/Bot/BotController.cs`

- [ ] **Шаг 1: BotController.cs**

```csharp
// Assets/Scripts/Bot/BotController.cs
using UnityEngine;
using System.Collections.Generic;

public class BotController : MonoBehaviour
{
    public List<UnitBase> botUnitPool;
    public GridManager gridManager;
    public int botUnitCount = 4;
    public int botRows = 2; // бот занимает верхние 2 ряда

    public void SpawnBotUnits()
    {
        int gridRows = gridManager.rows;
        List<GridCell> botCells = new();

        for (int r = gridRows - botRows; r < gridRows; r++)
            for (int c = 0; c < gridManager.cols; c++)
            {
                var cell = gridManager.GetCell(r, c);
                if (cell.IsEmpty) botCells.Add(cell);
            }

        Shuffle(botCells);
        int count = Mathf.Min(botUnitCount, botCells.Count);

        for (int i = 0; i < count; i++)
        {
            int idx = Random.Range(0, botUnitPool.Count);
            var unit = Instantiate(botUnitPool[idx],
                botCells[i].transform.position, Quaternion.identity);
            unit.teamId = 1;
            unit.currentCell = botCells[i];
            botCells[i].occupant = unit;
        }
    }

    private void Shuffle<T>(List<T> list)
    {
        for (int i = list.Count - 1; i > 0; i--)
        {
            int j = Random.Range(0, i + 1);
            (list[i], list[j]) = (list[j], list[i]);
        }
    }
}
```

- [ ] **Шаг 2: Подключить в сцену через MCP**

```
Добавь BotController на объект GameManager в сцене. Заполни botUnitPool через Inspector (все 3 архетипа). Вызови SpawnBotUnits() перед StartBattle().
```

- [ ] **Шаг 3: Проверить полный цикл**

Запустить игру → бот расставляет юнитов → нажать «НАЧАТЬ БОЙ» → юниты дерутся → кто-то побеждает.

- [ ] **Шаг 4: Коммит**
```bash
git add Assets/Scripts/Bot/
git commit -m "feat: bot controller that auto-places enemy units"
git push origin main
```

---

## Task 9: Первые 3D ассеты (placeholder → реальные)

**Цель:** заменить capsule-заглушки на реальные модели через Meshy.ai.

**Файлы:**
- Create: `Assets/Models/Sportik.fbx`
- Create: `Assets/Models/Otmorozok.fbx`
- Create: `Assets/Models/Vyezdnoy.fbx`
- Create: `Assets/Animations/Idle.anim`, `Assets/Animations/Attack.anim`, `Assets/Animations/Death.anim`

- [ ] **Шаг 1: Сгенерировать модели в Meshy.ai**

На [meshy.ai](https://meshy.ai) → Image to 3D:
- Загрузить референс парня в спортивных штанах и футболке
- Сгенерировать 3 варианта (для каждого архетипа)
- Экспортировать `.fbx` или `.glb`

- [ ] **Шаг 2: Скачать анимации с Mixamo**

На [mixamo.com](https://mixamo.com):
- Загрузить модель (Upload Character)
- Скачать: `Idle`, `Punch`, `Falling Back Death`, `Running`
- Формат: FBX for Unity

- [ ] **Шаг 3: Импортировать в Unity**

Перетащить `.fbx` файлы в `Assets/Models/`. Создать Animator Controller для каждого архетипа.

- [ ] **Шаг 4: Подключить к префабам юнитов**

Заменить Capsule mesh на импортированную модель. Добавить Animator компонент.

- [ ] **Шаг 5: Коммит**
```bash
git add Assets/Models/ Assets/Animations/
git commit -m "feat: replace placeholder capsules with 3D character models"
git push origin main
```

---

## Task 10: Полировка демо и финальная проверка

**Цель:** демо должно пройти полный цикл без ошибок: выбор → покупка → расстановка → бой → результат.

- [ ] **Шаг 1: Добавить экран результата**

```csharp
// Добавить в BattleManager:
public TMP_Text resultText; // подключить в Inspector

// В Update(), когда бой заканчивается:
resultText.text = team0 ? "ВАША ФИРМА ПОБЕДИЛА" : "ПОРАЖЕНИЕ";
resultText.gameObject.SetActive(true);
```

- [ ] **Шаг 2: Добавить базовый звук**

В Unity Asset Store найти бесплатный пак crowd/fight звуков. Добавить AudioSource на BattleManager, воспроизводить при атаке и победе.

- [ ] **Шаг 3: Финальный playthrough**

Пройти полный цикл 3 раза подряд без крашей или ошибок в Console.

- [ ] **Шаг 4: Финальный коммит**
```bash
git add .
git commit -m "feat: demo complete - full game loop working"
git push origin main
```

---

## Верификация (как проверить что демо работает)

1. Запустить Unity → Play Mode
2. В магазине появляются юниты (RefreshShop)
3. Покупка юнита снимает золото
4. Юнит появляется на сетке
5. Нажать «НАЧАТЬ БОЙ» → бот расставляет своих
6. Юниты движутся к врагам и атакуют
7. Отморозок иногда бьёт своих
8. Спортик иногда оглушает врага (AI не двигается 1.5с)
9. Выездной даёт бонус урона союзникам рядом
10. Бой заканчивается когда одна сторона выбита
11. Появляется текст результата

---

## Открытые вопросы (не блокируют демо, решить в v2)

- Тип сетки: hex или квадратная?
- Система фракций с бонусами (баланс)
- Полный список классов всех архетипов
- Экономика: процент на золото между раундами
- Анимации фоновых NPC (дед, мама с коляской, операторы)

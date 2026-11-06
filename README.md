```mermaid
flowchart TD
    Start(["Начало"]) --> Config["Настройки: источник — Исходка!G6,<br/>приёмник — ОЖР!J25,<br/>maxChars = 12"]
    Config --> GetText["Прочитать текст, Trim"]
    GetText --> CheckEmpty{"Текст пустой?"}
    CheckEmpty -- Да --> ShowMsgEmpty["MsgBox «Ячейка пуста»"] --> End(["Конец"])
    CheckEmpty -- Нет --> SplitWords["Split по пробелам → words"]
    SplitWords --> InitLine["currentLine = ''"]
    InitLine --> LoopWords["Цикл по словам:<br/>i = от LBound до UBound"]

    LoopWords --> GetWord["Взять очередное слово"]
    GetWord --> SkipEmpty{"Слово пустое?"}
    SkipEmpty -- Да --> NextWord["Следующее слово"]
    SkipEmpty -- Нет --> CheckLen{"Длина слова > maxChars?"}

    %% Ветка обычного слова
    CheckLen -- Нет --> TryAddToLine{"currentLine пустая?"}
    TryAddToLine -- Да --> SetLine["currentLine = word"]
    TryAddToLine -- Нет --> BuildTest["testLine = currentLine + пробел + word"]
    BuildTest --> CheckFit{"Длина testLine ≤ maxChars?"}
    CheckFit -- Да --> UpdateLine["currentLine = testLine"]
    CheckFit -- Нет --> WriteLine["Записать currentLine в ячейку,<br/>rowIndex + 1,<br/>currentLine = word"]
    UpdateLine --> NextWord
    WriteLine --> NextWord

    %% Ветка длинного слова
    CheckLen -- Да --> ReplaceDash["Заменить «—» на «-»"]
    ReplaceDash --> SplitParts["Split по дефису → parts"]
    SplitParts --> LoopParts["Цикл по частям"]

    LoopParts --> GetPart["Взять part = Trim части"]
    GetPart --> SkipPart{"Часть пустая?"}
    SkipPart -- Да --> NextPart["Следующая часть"]
    SkipPart -- Нет --> LastPart{"Это последняя часть?"}
    LastPart -- Да --> NoDash["partWithDash = part"]
    LastPart -- Нет --> AddDash["partWithDash = part + дефис"]
    AddDash --> NoDash
    NoDash --> CheckPartLen{"Длина partWithDash > maxChars?"}

    CheckPartLen -- Нет --> TryAddPart{"currentLine пустая?"}
    TryAddPart -- Да --> SetPartLine["currentLine = partWithDash"]
    TryAddPart -- Нет --> BuildPartTest["testLine = currentLine + пробел + partWithDash"]
    BuildPartTest --> CheckPartFit{"Длина testLine ≤ maxChars?"}
    CheckPartFit -- Да --> UpdatePartLine["currentLine = testLine"]
    CheckPartFit -- Нет --> WritePartLine["Записать currentLine,<br/>rowIndex + 1,<br/>currentLine = partWithDash"]
    UpdatePartLine --> NextPart
    WritePartLine --> NextPart

    CheckPartLen -- Да --> CallHyphen["Вызов AddLongPartWithHyphenation"]
    CallHyphen --> NextPart
    NextPart --> CheckPartsEnd{"Есть ещё части?"}
    CheckPartsEnd -- Да --> LoopParts
    CheckPartsEnd -- Нет --> NextWord
    NextWord --> CheckWordsEnd{"Есть ещё слова?"}
    CheckWordsEnd -- Да --> LoopWords
    CheckWordsEnd -- Нет --> WriteLast{"currentLine не пустая?"}
    WriteLast -- Да --> SaveLast["Записать последнюю currentLine"]
    WriteLast -- Нет --> ShowMsgDone["MsgBox «Текст перенесён…»"]
    SaveLast --> ShowMsgDone
    ShowMsgDone --> End

    subgraph Hyp["Подсхема: AddLongPartWithHyphenation"]
        HypStart(["Вход"]) --> SetRemaining["remaining = part"]
        SetRemaining --> WhileLoop{"Длина remaining > maxChars?"}
        WhileLoop -- Нет --> HandleRemaining{"remaining не пустой?"}
        HandleRemaining -- Нет --> HypEnd(["Выход"])
        HandleRemaining -- Да --> TryAddRem{"currentLine пустая?"}
        TryAddRem -- Да --> SetRemLine["currentLine = remaining"]
        TryAddRem -- Нет --> BuildRemTest["testLine = currentLine + пробел + remaining"]
        BuildRemTest --> CheckRemFit{"Длина testLine ≤ maxChars?"}
        CheckRemFit -- Да --> UpdateRemLine["currentLine = testLine"]
        CheckRemFit -- Нет --> WriteRemLine["Записать currentLine,<br/>rowIndex + 1,<br/>currentLine = remaining"]
        UpdateRemLine --> HypEnd
        WriteRemLine --> HypEnd

        WhileLoop -- Да --> FindPos["pos = FindHyphenationPos(remaining, maxChars)"]
        FindPos --> CheckPos{"pos = 0 — перенос невозможен?"}
        CheckPos -- Да --> HandleNoPos["Добавить remaining целиком:<br/>влезает → в строку,<br/>нет → записать и rowIndex + 1"] --> HypEnd
        CheckPos -- Нет --> MakeFirstPart["firstPart = Left(remaining, pos) + дефис"]
        MakeFirstPart --> TryAddFirst{"currentLine пустая?"}
        TryAddFirst -- Да --> SetFirstLine["currentLine = firstPart"]
        TryAddFirst -- Нет --> BuildFirstTest["testLine = currentLine + пробел + firstPart"]
        BuildFirstTest --> CheckFirstFit{"Длина testLine ≤ maxChars?"}
        CheckFirstFit -- Да --> UpdateFirstLine["currentLine = testLine"]
        CheckFirstFit -- Нет --> WriteFirstLine["Записать currentLine,<br/>rowIndex + 1,<br/>currentLine = firstPart"]
        UpdateFirstLine --> WriteCurrent["Записать currentLine,<br/>rowIndex + 1,<br/>currentLine = пустая"]
        WriteFirstLine --> WriteCurrent
        WriteCurrent --> UpdateRemaining["remaining = остаток после pos"]
        UpdateRemaining --> WhileLoop
    end

    subgraph FPos["Подсхема: FindHyphenationPos"]
        FStart(["Вход: word, maxChars"]) --> InitBest["bestPos = 0"]
        InitBest --> CalcLimit["limit = min(maxChars, длина слова − 1)"]
        CalcLimit --> LoopI["Цикл: i от limit до 2, шаг −1"]
        LoopI --> CheckI{"i < длина слова?"}
        CheckI -- Нет --> NextI["Следующий i"]
        CheckI -- Да --> CheckVowel{"Символ i — гласная?"}
        CheckVowel -- Нет --> NextI
        CheckVowel -- Да --> CheckCons{"Символ i−1 — согласная?"}
        CheckCons -- Нет --> NextI
        CheckCons -- Да --> SetBest["bestPos = i − 1"] --> ReturnResult["Результат = bestPos"]
        NextI --> CheckLoopEnd{"i достиг 2?"}
        CheckLoopEnd -- Нет --> LoopI
        CheckLoopEnd -- Да --> ReturnResult
        ReturnResult --> FEnd(["Выход"])
    end

    subgraph Vowel["Подсхема: IsVowel"]
        VStart(["Вход: символ ch"]) --> SetVowels["vowels = строка всех гласных"]
        SetVowels --> CheckInStr{"Символ найден в строке гласных?"}
        CheckInStr -- Да --> ReturnTrue["IsVowel = True"]
        CheckInStr -- Нет --> ReturnFalse["IsVowel = False"]
        ReturnTrue --> VEnd(["Выход"])
        ReturnFalse --> VEnd
    end

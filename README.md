mermaid```` 
flowchart TD
    Start([Начало]) --> Config[Настройки:<br/>Источник: Исходка!G6<br/>Приёмник: ОЖР!J25<br/>maxChars = 12]
    Config --> GetText[Прочитать текст, Trim]
    GetText --> CheckEmpty{Текст пустой?}
    CheckEmpty -- Да --> ShowMsgEmpty[MsgBox «Ячейка пуста»] --> End([Конец])
    CheckEmpty -- Нет --> SplitWords[Split текста по пробелам → words]
    SplitWords --> InitLine[currentLine = ""]
    InitLine --> LoopWords[Цикл по словам: i = LBound(words) … UBound(words)]
    
    LoopWords --> GetWord[Взять word = words[i]]
    GetWord --> SkipEmpty{word = ""?}
    SkipEmpty -- Да --> NextWord[Следующее слово]
    SkipEmpty -- Нет --> CheckLen{Len(word) > maxChars?}
    
    %% Ветка обычного слова
    CheckLen -- Нет --> TryAddToLine{currentLine = ""?}
    TryAddToLine -- Да --> SetLine[currentLine = word]
    TryAddToLine -- Нет --> BuildTest[testLine = currentLine & " " & word]
    BuildTest --> CheckFit{Len(testLine) ≤ maxChars?}
    CheckFit -- Да --> UpdateLine[currentLine = testLine]
    CheckFit -- Нет --> WriteLine[Записать currentLine в ячейку<br/>rowIndex + 1<br/>currentLine = word]
    UpdateLine --> NextWord
    WriteLine --> NextWord
    
    %% Ветка длинного слова
    CheckLen -- Да --> ReplaceDash[Заменить «—» на «-»]
    ReplaceDash --> SplitParts[Split по «-» → parts]
    SplitParts --> LoopParts[Цикл по частям: j = LBound(parts) … UBound(parts)]
    
    LoopParts --> GetPart[Взять part = Trim(parts[j])]
    GetPart --> SkipPart{part = ""?}
    SkipPart -- Да --> NextPart[Следующая часть]
    SkipPart -- Нет --> LastPart{j = UBound(parts)?}
    LastPart -- Да --> NoDash[partWithDash = part]
    LastPart -- Нет --> AddDash[partWithDash = part & "-"]
    AddDash --> NoDash
    NoDash --> CheckPartLen{Len(partWithDash) > maxChars?}
    
    CheckPartLen -- Нет --> TryAddPart{currentLine = ""?}
    TryAddPart -- Да --> SetPartLine[currentLine = partWithDash]
    TryAddPart -- Нет --> BuildPartTest[testLine2 = currentLine & " " & partWithDash]
    BuildPartTest --> CheckPartFit{Len(testLine2) ≤ maxChars?}
    CheckPartFit -- Да --> UpdatePartLine[currentLine = testLine2]
    CheckPartFit -- Нет --> WritePartLine[Записать currentLine<br/>rowIndex + 1<br/>currentLine = partWithDash]
    UpdatePartLine --> NextPart
    WritePartLine --> NextPart
    
    CheckPartLen -- Да --> CallHyphen[Call AddLongPartWithHyphenation]
    CallHyphen --> NextPart
    NextPart --> CheckPartsEnd{j < UBound(parts)?}
    CheckPartsEnd -- Да --> LoopParts
    CheckPartsEnd -- Нет --> NextWord
    NextWord --> CheckWordsEnd{i < UBound(words)?}
    CheckWordsEnd -- Да --> LoopWords
    CheckWordsEnd -- Нет --> WriteLast{currentLine ≠ ""?}
    WriteLast -- Да --> SaveLast[Записать последнюю currentLine]
    WriteLast -- Нет --> ShowMsgDone[MsgBox «Текст перенесён…»]
    SaveLast --> ShowMsgDone
    ShowMsgDone --> End

    subgraph Hyphenation["Подсхема: AddLongPartWithHyphenation"]
        HypStart([Вход]) --> SetRemaining[remaining = part]
        SetRemaining --> WhileLoop{Len(remaining) > maxChars?}
        WhileLoop -- Нет --> HandleRemaining{remaining ≠ ""?}
        HandleRemaining -- Да --> TryAddRem{currentLine = ""?}
        TryAddRem -- Да --> SetRemLine[currentLine = remaining]
        TryAddRem -- Нет --> BuildRemTest[testLine3 = currentLine & " " & remaining]
        BuildRemTest --> CheckRemFit{Len(testLine3) ≤ maxChars?}
        CheckRemFit -- Да --> UpdateRemLine[currentLine = testLine3]
        CheckRemFit -- Нет --> WriteRemLine[Записать currentLine<br/>rowIndex + 1<br/>currentLine = remaining]
        UpdateRemLine --> HypEnd([Выход])
        WriteRemLine --> HypEnd
        
        WhileLoop -- Да --> FindPos[pos = FindHyphenationPos(remaining, maxChars)]
        FindPos --> CheckPos{pos = 0?}
        CheckPos -- Да --> HandleNoPos[Добавить remaining целиком:<br/>влезает → currentLine,<br/>нет → записать, rowIndex+1]
        HandleNoPos --> HypEnd
        CheckPos -- Нет --> MakeFirstPart[firstPart = Left(remaining, pos) & "-"]
        MakeFirstPart --> TryAddFirst{currentLine = ""?}
        TryAddFirst -- Да --> SetFirstLine[currentLine = firstPart]
        TryAddFirst -- Нет --> BuildFirstTest[testLine4 = currentLine & " " & firstPart]
        BuildFirstTest --> CheckFirstFit{Len(testLine4) ≤ maxChars?}
        CheckFirstFit -- Да --> UpdateFirstLine[currentLine = testLine4]
        CheckFirstFit -- Нет --> WriteFirstLine[Записать currentLine<br/>rowIndex + 1<br/>currentLine = firstPart]
        UpdateFirstLine --> WriteCurrent[Записать currentLine<br/>rowIndex + 1<br/>currentLine = ""]
        WriteFirstLine --> WriteCurrent
        WriteCurrent --> UpdateRemaining[remaining = Mid(remaining, pos + 1)]
        UpdateRemaining --> WhileLoop
    end

    subgraph FindPosSub["Подсхема: FindHyphenationPos"]
        FStart([Вход: word, maxChars]) --> InitBest[bestPos = 0]
        InitBest --> CalcLimit[limit = min(maxChars, Len(word)-1)]
        CalcLimit --> LoopI[Цикл: i = limit … 2, шаг -1]
        LoopI --> CheckI{i < Len(word)?}
        CheckI -- Нет --> NextI[Следующий i]
        CheckI -- Да --> CheckVowel{Символ i — гласная?}
        CheckVowel -- Нет --> NextI
        CheckVowel -- Да --> CheckCons{Символ i-1 — согласная?}
        CheckCons -- Нет --> NextI
        CheckCons -- Да --> SetBest[bestPos = i - 1] --> ExitLoop[Exit For]
        ExitLoop --> ReturnResult[FindHyphenationPos = bestPos]
        NextI --> CheckLoopEnd{i > 2?}
        CheckLoopEnd -- Да --> LoopI
        CheckLoopEnd -- Нет --> ReturnResult
    end

    subgraph IsVowelSub["Подсхема: IsVowel"]
        VStart([Вход: ch]) --> SetVowels[vowels = строка всех гласных]
        SetVowels --> CheckInStr{InStr(vowels, ch) > 0?}
        CheckInStr -- Да --> ReturnTrue[IsVowel = True]
        CheckInStr -- Нет --> ReturnFalse[IsVowel = False]
        ReturnTrue --> VEnd([Выход])
        ReturnFalse --> VEnd
    end

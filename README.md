# -prompt_engineering
Chain of Thought and Self-Consistency Chain of Thought implementation on GSM8K with bigscience/bloom-7b1-petals

**Инструкция по коду:** в репозитории лежит ноутбук prompt_test, в комментариях даны объяснения к каждой функции / блоку. Запускать по порядку. 

В данном репозитории реализовано 2 метода в области prompt engineering – Chain of Thought and Self-Consistency Chain of Thought. 
Прежде чем описывать результаты, важно отметить, что работа выполнена с использованием модели "bigscience/bloom-7b1-petals" (а не "bigscience/bloom 176") из-за ограничений в работе bloom, также это уточнялось в дискорде. Кроме того, удалось посчитать только на выборке из 20 примеров для каждого вида данных (об этом чуть ниже), в связи с ограничениями по использованию ГПУ в Колабе. В будущих работах необходимо заранее ставить эксперименты, чтобы избежать подобных затруднений, однако сейчас данный код может использоваться как структура для будущих исследований prompt engineering. 

Итак, помимо реализации двух методов prompt engineering было проведено разделение на а) данные с mathematical equation (16 + 4 = 20) и без него  и б) на фиксированный (одинаковый для примеров) набор промптов и уникальный набор для каждого примера из трейна. В связи с тем, что датасет GSM8K уже включает в себя reasoning steps, можно использовать их как промпты (каждый раз семплить разные или оставлять одинаковыми для примеров из теста). Было выбрано 8 промптов, как в статье https://arxiv.org/abs/2201.11903. В кейсе Self-Consistency было сгенерировано по 5 примеров на каждый тест из теста. Возможно в будущем стоит перебрать данные два параметра и сравнить результаты.   

**Результаты:** 
В выборке из 20 примеров для каждого набора данных и каждого метода только в одном кейсе с фиксированными промптами и greedy декодингом (классический Chain of Thought) был получен правильный ответ. Итого, можно говорить о нулевой точности во всех слуачях, кроме классического Chain of Thought (5%). Было также выявлено, что в почти в среднем по всем наборам данных в 49.2% случаев (минимум 0.3 в классическом Chain of Thought с фиксированными/разными промптами, максимум 0.7 в Self-Consistency с различными промптами) модель вообще не предсказывает ответ (не выводит The answer is:). Возможно, увеличение набора промптов или же другая формулировка ответа может снизить долю UNK в данных. Трудно сранивать результаты с другими моделям (даже с UL2 как наиболее близкой по размеру моделью) в связи с небольшим количеством примеров, однако даже на полном объеме тестовых данных в кейсе UL2 доля правильных ответов достаточно низкая (4.1 Chain of Thought and 7.3 Self-Consistency Chain of Thought). 

**Как улучшить результаты?** 
Безусловно, попробовать еще раз реализовать код на а) всех данных и б) с использованием большой модели :) 

Основная идея для улучшения: недавно вышла статья про Backward Chaining – идея в том, что мы отталкиваемся от финального ответа, идем с помощью алгоритма по всем шагам и доходим до искомого утверждения, которое подтверждается или опровергается (подробнее https://arxiv.org/pdf/2212.13894.pdf). Это натолкнуло меня на идею, что можно сгенерировать ансамбль из "обратных" промптов (вопрос + ответ + размышления почему так) и "классических Chain of Thought" и, например, смотреть на моду в пересечениях двух ансамблей. Кроме того, в ноушене представлена идея с обучением модели с использованием комбинации Backward Chaining и Chain of Thought https://massive-search-72b.notion.site/Research-proposal-b4bb365b75e84ad48950099334c08603

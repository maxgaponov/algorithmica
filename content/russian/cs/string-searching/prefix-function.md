---
title: Префикс-функция
authors:
- Сергей Слотин
created: 2018
weight: 1
---

**Определение**. Префикс-функцией от строки $s$ называется массив $p$, где $p_i$ равно длине самого большого префикса строки $s_0 s_1 s_2 \ldots s_i$, который также является и суффиксом $i$-того префика (не считая весь $i$-й префикс).

Например, самый большой префикс, который равен суффиксу для строки «aataataa» — это «aataa»; префикс-функция для этой строки равна $[0, 1, 0, 1, 2, 3, 4, 5]$.

```c++
vector<int> slow_prefix_function(string s) {
    int n = (int) s.size();
    vector<int> p(n, 0);
    for (int i = 1; i < n; i++)
        for (int len = 1; len <= i; len++)
            // если префикс длины len равен суффиксу длины len
            if (s.substr(0, len) == s.substr(i - len + 1, len))
                p[i] = len;
    return p;
}
```

Этот алгоритм пока что работает за $O(n^3)$, но позже мы его ускорим.

### Как это поможет решить исходную задачу?

Давайте пока поверим, что мы умеем считать префикс-функцию за линейное от размера строки, и научимся с помощью нее искать подстроку в строке.

Соединим подстроки $s$ и $t$ каким-нибудь символом, который не встречается ни там, ни там — обозначим пусть этот символ `#`. Посмотрим на префикс-функцию получившейся строки `s#t`.

```c++
string s = "choose";
string t =
    "choose life. choose a job. choose a career. choose a family. choose a fu...";

cout << s + "#" + t << endl;
cout << slow_prefix_function(s + "#" + t) << endl;
```

```bash
choose#choose life. choose a job. choose a career. choose a family. choose a fu...
0000000123456000000012345600000000123456000100000001234560000000000012345600000000
```

Видно, что все места, где значения равны 6 (длине $s$) — это концы вхождений $s$ в текст $t$.

Такой алгоритм (посчитать префикс-функцию от `s#t` и посмотреть, в каких позициях она равна $|s|$) называется **алгоритмом Кнута-Морриса-Пратта**.

### Как её быстро считать

Рассмотрим ещё несколько примеров префикс-функций и попытаемся найти закономерности:

```bash
aaaaa
01234

abcdef
000000

abacabadava
00101230101
```

Можно заметить следующую особенность: $p_{i+1}$ максимум на единицу превосходит $p_i$.

**Доказательство.** Если есть префикс, равный суффиксу строки $s_{:i+1}$, длины $p_{i+1}$, то, отбросив последний символ, можно получить правильный суффикс для строки $s_{:i}$, длина которого будет ровно на единицу меньше.

Попытаемся решить задачу с помощью динамики: найдём формулу для $p_i$ через предыдущие значения.

Заметим, что $p_{i+1} = p_i + 1$ в том и только том случае, когда $s_{p_i} =s_{i+1}$. В этом случае мы можем просто обновить $p_{i+1}$ и пойти дальше.

Например, в строке $\underbrace{aabaa}t\overbrace{aabaa}$ выделен максимальный префикс, равный суффиксу: $p_{10} = 5$. Если следующий символ равен будет равен $t$, то $p_{11} = p_{10} + 1 = 6$.

Но что происходит, когда $s_{p_i}\neq s_{i+1}$? Пусть следующий символ в этом же примере равен не $t$, а $b$.

* $\implies$ Длина префикса, равного суффиксу новой строки, будет точно меньше 5.
* $\implies$ Помимо того, что искомый новый супрефикс является суффиксом «aabaa**b**», он ещё является префиксом подстроки «aabaa».
* $\implies$ Значит, следующий кандидат на проверку — это значение префикс-функции от «aabaa», то есть $p_4 = 2$, которое мы уже посчитали.
* $\implies$ Если $s_2 = s_{11}$ (т. е. новый символ совпадает с идущим после префикса-кандидата), то $p_{11} = p_2 + 1 = 2 + 1 = 3$.

В данном случае это действительно так (нужный префикс — «aab»). Но что делать, если, в общем случае, $p_{i+1} \neq p_{p_i+1}$? Тогда мы проводим такое же рассуждение и получаем нового кандидата, меньшей длины — $p_{p_{p_i}}$. Если и этот не подошел — аналогично проверяем меньшего, пока этот индекс не станет нулевым.

```c++
vector<int> prefix_function(string s) {
    int n = (int) s.size();
    vector<int> p(n, 0);
    for (int i = 1; i < n; i++) {
        // префикс функция точно не больше этого значения + 1
        int cur = p[i - 1];
        // уменьшаем cur значение, пока новый символ не сматчится
        while (s[i] != s[cur] && cur > 0)
            cur = p[cur - 1];
        // здесь либо s[i] == s[cur], либо cur == 0
        if (s[i] == s[cur])
            p[i] = cur + 1;
    }
    return p;
}
```

**Асимптотика.** В худшем случае этот `while` может работать $O(n)$ раз за одну итерацию, но *в среднем* каждый `while` работает за $O(1)$.

Префикс-функция каждый шаг возрастает максимум на единицу и после каждой итерации `while` уменьшается хотя бы на единицу. Значит, суммарно операций будет не более $O(n)$.
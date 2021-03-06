# Отчет по курсовому проекту
## по курсу "Логическое программирование"

### студент: Макаров В.А.

## Результат проверки

| Преподаватель     | Дата         |  Оценка       |
|-------------------|--------------|---------------|
| Сошников Д.В. |              |               |
| Левинская М.А.|              |      5-       |

> *Комментарии проверяющих (обратите внимание, что более подробные комментарии возможны непосредственно в репозитории по тексту программы)*

## Введение

 В результате выполнения курсового проекта можно получить множество навыков и знаний. Я познакомлюсь с языком prolog, и некоторыми его возможностями. Я научусь составлять и обрабатывать предикаты, работать со списками, решать логические задачи, выполнять поиск решений в графе и обрабатывать запросы на естественном языке. Я узнаю, почему prolog хорош для выполнения этих задач и на практике увижу преимущества этого языка. Более подробно об этом будет указано в выводах. 

## Задание

 1. Создать родословное дерево своего рода на несколько поколений (3-4) назад в стандартном формате GEDCOM с использованием сервиса MyHeritage.com 
 2. Преобразовать файл в формате GEDCOM в набор утверждений на языке Prolog, используя следующее представление: ...
 3. Реализовать предикат проверки/поиска .... 
 4. Реализовать программу на языке Prolog, которая позволит определять степень родства двух произвольных индивидуумов в дереве
 5. [На оценки хорошо и отлично] Реализовать естественно-языковый интерфейс к системе, позволяющий задавать вопросы относительно степеней родства, и получать осмысленные ответы. 

## Получение родословного дерева

Для получения родословного дерева я использовал сайт myheritage.ru, который преобразует информацию из интуитивно-понятной визуальной формы в формат GEDCOM. В моем дереве получилось 42 человека. Часть людей, информации о которых не нашлось, были заменены вымышленными именами.

## Конвертация родословного дерева

Для обработки GEDCOM-файла я использовал язык python. Выполнение парсинга на других языках программирования также возможно, но я выбрал удобный для составления короткой и лакончиной программы язык. Для решения этой задачи я использовал готовые библиотеки python-gedcom, находящиеся в свободном доступе в интернете.

главная часть кода:

```python
for element in root_child_elements:
if isinstance(element, IndividualElement):
if len(gedcom_parser.get_parents(element, "ALL")) != 0:
(father, mother) = gedcom_parser.get_parents(element, "ALL")
(child_first_name, child_last_name) = element.get_name()
(father_first_name, father_last_name) = father.get_name()
(mother_first_name, mother_last_name) = mother.get_name()
facts.write('parents(' + child_first_name + ' ' + child_last_name + ',' + father_first_name + ' ' + father_last_name + ', ' + mother_first_name + ' ' + mother_last_name + ')\n')
```
Основной принцип работы следующий - при помощи стандартных методов библиотеки (root_child_elements - список всех потомков, get_parents - получение родителей конкретного потомка, get_name - получение имени, и.т.д.) проходимся по всем потомкам в списке, получая информацию о его родителях, и записываем полученную информацию в итоговый файл в необходимом виде, т.е. в форме фактов для языка prolog.
Я выбрал такой способ решения, поскольку данная библиотека обладает всеми необходимыми методами для парсинга GEDCOM файла, позволяя сделать программу интуитивно-понятной.
Информация о библиотеке:
https://pypi.org/project/python-gedcom/0.2.5.dev0/

## Предикат поиска родственника

Предикат поиска двоюродного брата.

Поскольку мы имеем факты в форме parents(child,father,mother) удобнее и эффективнее всего будет использовать дополнительные предикаты (они пригодятся и в последующих заданиях).

Предикат cild(Name,X) // ищет родителей (Name) для выбранного потомка (X) 
Мы можем найти их по местоположению в описанных фактах.

```prolog
child(Name,X) :- parents(Name, X, _ ); parents(Name, _ , X).
```

Предикат sibling(Name,X) // ищет братьев/сестер (Name) для выбранного человека (X)
Найти их можно, сопоставив всех потомков, имеющих одних и тех же родителей.

```prolog
sibling(Name,X) :-parents(Name, A,B), parents(X, A, B),
Name \= X.
```
Главный предикат: cousin_bro(Name,X) // ищет двоюродных братьев (Name) для выбранного человека (X)
При реализации предиката я столкнулся с проблемой: поскольку родство у нас задано предикатом parents(child, father, mother), у нас нет информации о том, какого пола тот или иной объект child. Единственный способ узнать пол - если у этого потомка есть свои дети, тогда мы можем определить пол, узнав отец это или мать. Но если у потомка нет своих детей, то есть как минимум 3 варианта решения этой проблемы: введение дополнительного предиката для пола, добавление фиктивных потомков или игнорирование объектов, не имеющих потомков. 
Я думаю, что, в общем случае,самым полным и правильным способом было бы введение дополнительного предиката для пола, но посмотрев на свой конкретный пример, я заметил, что у всех, кто мог бы быть кому-либо двоюродным братом, за редким исключением, есть потомки. Поэтому, разрабатывая решение для своей конкретной задачи я выбрал третий вариант - игнорировать тех, у кого нет потомков. Для этого добавляем условие parents(_ ,Name,_ ) - кандидат является кому-то отцом. 

```prolog
cousin_bro(Name,X) :- ((sibling(A,B), child(Name,A), child(X,B),parents(_ ,Name,_ ));
(sibling(A,B), child(X,A), child(Name,B), parents(_ ,Name,_ ))),
Name \= X.
```
Пример использования:

```prolog
parents('Александр Макаров','Борис Макаров', 'Елизавета Макарова').
parents('Валерий Макаров','Александр Макаров', 'Елена Макарова').
parents('Елена Макарова','Кузьма Титов', 'Вера Титова').
parents('Борис Макаров','Алексей Макаров', 'Светлана Макарова').
parents('Алексей Макаров','Вячеслав Макаров', 'Ольга Макарова').
parents('Вячеслав Макаров','Виктор Макаров', 'Виктория Макарова').
parents('Владимир Макаров','Борис Макаров', 'Елизавета Макарова').
parents('Сергей Макаров','Владимир Макаров', 'Александра Мышляева').
parents('Антон Макаров','Алексей Макаров', 'Светлана Макарова').
parents('Иван Макаров','Антон Макаров', 'Анастасия Акчурина').
parents('Анастасия Акчурина','Ренат Акчурин', 'Анна Акчурина').
parents('Анна Акчурина','Артур Акчурин', 'Лера Акчурина').
parents('Игорь Акчурин','Артур Акчурин', 'Лера Акчурина').
parents('Олег Акчурин','Игорь Акчурин','Яна Петрова').
parents('Григорий Макаров','Виктор Макаров', 'Виктория Макарова').
parents('Алексей Макаров','Григорий Макаров', 'Анна Пономарева').
parents('Вера Титова','Александр Володьков', 'Татьяна Володькова').
parents('Татьяна Володькова','Анатолий Володьков', 'Анна Володькова').
parents('Ольга Володькова','Анатолий Володьков', 'Анна Володькова').
parents('Аркадий Володьков','Павел Володьков', 'Ольга Володькова').
parents('Алиса Макарова','Владимир Макаров', 'Александра Мышляева').
parents('Иван Володьков','Александр Володьков', 'Татьяна Володькова').
parents('Федор Макаров','Сергей Макаров', 'Юлия Макарова').
parents('Арсений Володьков','Иван Володьков', 'Людмила Володькова').


?- child(Name, 'Александр Макаров').
Name = 'Валерий Макаров'

?- sibling(Name, 'Александр Макаров').
Name = 'Владимир Макаров'

?- child(Name, 'Владимир Макаров').
Name = 'Сергей Макаров'
Name = 'Алиса Макарова'

?- cousin_bro(Name, 'Валерий Макаров').
Name = 'Сергей Макаров'

?- cousin_bro('Алиса Макарова','Валерий Макаров').
false

?- cousin_bro('Сергей Макаров','Валерий Макаров').
true
```


## Определение степени родства

Сначала опишем предикат, задающий стандартные (ребенок, отец, мать, брат/сестра, двоюродный брат/сестра, жена, муж) отношения между двумя людьми, из которого можно будет извлечь и само "название отношения". Для этого просто добавим название отношения третим аргументом:

```prolog
relative(child, A, B) :- parents(A, _, B); parents(A, B, _).    
relative(father, A, B) :- parents(B, A, _).
relative(mother, A, B) :- parents(B, _, A).
relative(husband, A, B) :- parents(_, A, B), !.
relative(wife, A, B) :- parents(_, A, B), !.
relative(sibling, A, B) :- sibling(A, B).
relative(cousin, A, B) :- cousin(A, B).
```
Эти  предикаты послужат вершинами графа, по которому будет выполняться поиск.
Для решения данной задачи я решил использовать алгоритм поиска в ширину, полскольку именно этот алгоритм дает самый короткий путь, а затем ищет более длинные пути, что и требуется в данной задаче. 
Сам алгоритм был рассмотрен в 3 лабораторной работе, и почти не изменен.
Продвижение по графу:
```prolog
prolong([X|T],[Y,X|T]):-
	relative(_,X,Y),
	not(member(Y,[X|T])).
```
Поиск путей (алгоритм BFS):
```prolog
bfs([[H|T]|_],H,[H|T]).
bfs([Curr|Other],H,Way):-
	findall(W,prolong(Curr,W),Ways),
	append(Other,Ways,New), !,
	bfs(New,H,Way).
```
Добавление в список. Предикат получает на вход список имен, затем берет первые 2 элемента списка, ищет отношение между ними и записывает его в итоговый список, затем удаляет первый элемент.
```prolog
add([_], Obj, Obj).
add([X,Y|T], Obj, Relation) :-
    relative(Relate, X, Y),
    add([Y|T], [Relate|Obj], Relation). 
```
Запрос для поиска отношений (выполняем поиск всех возможных комбинаций):
```prolog
related(Relation, X, Y) :-
    bfs([[X]], Y, R),
    reverse(R, Chain),
    add(Chain, [], Relation1),
    reverse(Relation1, Relation).
```
Список отношений для поиска:
```prolog
relations(X):- member(X, [father, mother, sibling, husband, wife, cousin, child]).
```
Пример работы:
```prolog
?- related(X,'Борис Макаров','Валерий Макаров').       % дедушка

X = [father, father]                      % отец отца
X = [father, husband, mother]             % отец мужа матери   
X = [father, father, cousin]              % отец отца двоюродного брата
X = [husband, mother, father]             % и.т.д.
X = [father, child, mother, father]
X = [father, husband, mother, cousin]
X = [husband, mother, husband, mother]
X = [husband, mother, father, cousin]
X = [child, mother, father, cousin, father]
X = [child, father, father, cousin, father]
X = [father, child, mother, father, cousin]
X = [father, child, mother, husband, mother]
X = [father, father, child, mother, cousin]
X = [husband, mother, husband, mother, cousin]
X = [child, mother, father, cousin, husband, mother]
X = [child, mother, father, cousin, father, cousin]
X = [child, mother, husband, mother, cousin, father]
X = [child, father, father, cousin, husband, mother]
X = [child, father, father, cousin, father, cousin]
X = [child, father, husband, mother, cousin, father]
X = [child, husband, mother, father, cousin, father]
X = [father, child, mother, husband, mother, cousin]
X = [husband, mother, father, child, mother, cousin]
X = [child, mother, father, cousin, child, mother, father]
X = [child, mother, father, cousin, husband, mother, cousin]
X = [child, mother, husband, mother, cousin, husband, mother]
X = [child, mother, husband, mother, cousin, father, cousin]
X = [child, father, father, cousin, child, mother, father]
X = [child, father, father, cousin, husband, mother, cousin]
X = [child, father, husband, mother, cousin, husband, mother]
X = [child, father, husband, mother, cousin, father, cousin]
X = [child, husband, mother, father, cousin, husband, mother]
X = [child, husband, mother, father, cousin, father, cousin]
X = [child, husband, mother, husband, mother, cousin, father]
X = [father, child, mother, father, child, mother, cousin]
X = [child, mother, father, cousin, child, mother, father, cousin]
X = [child, mother, father, cousin, child, mother, husband, mother]
X = [child, mother, father, cousin, father, child, mother, cousin]
X = [child, mother, husband, mother, cousin, child, mother, father]
X = [child, mother, husband, mother, cousin, husband, mother, cousin]
X = [child, father, father, cousin, child, mother, father, cousin]
X = [child, father, father, cousin, child, mother, husband, mother]
X = [child, father, father, cousin, father, child, mother, cousin]
X = [child, father, husband, mother, cousin, child, mother, father]
X = [child, father, husband, mother, cousin, husband, mother, cousin]
X = [child, husband, mother, father, cousin, child, mother, father]
X = [child, husband, mother, father, cousin, husband, mother, cousin]
X = [child, husband, mother, husband, mother, cousin, husband, mother]
X = [child, husband, mother, husband, mother, cousin, father, cousin]
X = [child, mother, father, cousin, child, mother, husband, mother, cousin]
X = [child, mother, husband, mother, cousin, child, mother, father, cousin]
X = [child, mother, husband, mother, cousin, child, mother, husband, mother]
X = [child, mother, husband, mother, cousin, father, child, mother, cousin]
X = [child, father, father, cousin, child, mother, husband, mother, cousin]
X = [child, father, husband, mother, cousin, child, mother, father, cousin]
X = [child, father, husband, mother, cousin, child, mother, husband, mother]
X = [child, father, husband, mother, cousin, father, child, mother, cousin]
X = [child, husband, mother, father, cousin, child, mother, father, cousin]
X = [child, husband, mother, father, cousin, child, mother, husband, mother]
X = [child, husband, mother, father, cousin, father, child, mother, cousin]
X = [child, husband, mother, husband, mother, cousin, child, mother, father]
X = [child, husband, mother, husband, mother, cousin, husband, mother, cousin]
...


?- related(X,'Валерий Макаров','Сергей Макаров').
X = [cousin]
X = [cousin, sibling]
X = [cousin, child, mother]
X = [cousin, child, father]
X = [child, child, mother, father]
X = [child, child, father, father]
X = [cousin, child, husband, mother]
X = [child, child, mother, father, sibling]
X = [child, child, mother, husband, mother]
X = [child, child, father, father, sibling]
X = [child, child, father, husband, mother]
X = [child, child, husband, mother, father]
X = [child, child, mother, father, child, mother]
X = [child, child, mother, husband, mother, sibling]
X = [child, child, father, father, child, mother]
X = [child, child, father, husband, mother, sibling]
X = [child, child, husband, mother, father, sibling]
X = [child, child, husband, mother, husband, mother]
X = [child, child, child, mother, father, cousin, father]
X = [child, child, child, father, father, cousin, father]
X = [child, child, husband, mother, father, child, mother]
X = [child, child, husband, mother, husband, mother, sibling]
X = [child, child, child, mother, father, cousin, father, sibling]
X = [child, child, child, mother, father, cousin, husband, mother]
X = [child, child, child, mother, husband, mother, cousin, father]
X = [child, child, child, father, father, cousin, father, sibling]
X = [child, child, child, father, father, cousin, husband, mother]
X = [child, child, child, father, husband, mother, cousin, father]
X = [child, child, child, husband, mother, father, cousin, father]
X = [child, child, child, mother, father, cousin, father, child, mother]
X = [child, child, child, mother, father, cousin, husband, mother, sibling]
X = [child, child, child, mother, husband, mother, cousin, father, sibling]
X = [child, child, child, mother, husband, mother, cousin, husband, mother]
X = [child, child, child, father, father, cousin, father, child, mother]
X = [child, child, child, father, father, cousin, husband, mother, sibling]
X = [child, child, child, father, husband, mother, cousin, father, sibling]
X = [child, child, child, father, husband, mother, cousin, husband, mother]
X = [child, child, child, husband, mother, father, cousin, father, sibling]
X = [child, child, child, husband, mother, father, cousin, husband, mother]
X = [child, child, child, husband, mother, husband, mother, cousin, father]
X = [child, child, child, mother, husband, mother, cousin, father, child, mother]
...


?- related(X,'Александр Макаров','Сергей Макаров').
X = [father, cousin]
X = [child, mother, father]
X = [child, father, father]
X = [father, cousin, sibling]
X = [husband, mother, cousin]
X = [child, mother, father, sibling]
X = [child, mother, husband, mother]
X = [child, father, father, sibling]
X = [child, father, husband, mother]
X = [child, husband, mother, father]
X = [father, cousin, child, mother]
X = [father, cousin, child, father]
X = [husband, mother, cousin, sibling]
X = [child, mother, father, child, mother]
X = [child, mother, father, cousin, cousin]
X = [child, mother, husband, mother, sibling]
X = [child, father, father, child, mother]
X = [child, father, father, cousin, cousin]
X = [child, father, husband, mother, sibling]
X = [child, husband, mother, father, sibling]
X = [child, husband, mother, husband, mother]
X = [father, cousin, child, husband, mother]
X = [husband, mother, cousin, child, mother]
X = [husband, mother, cousin, child, father]
X = [child, mother, husband, mother, cousin, cousin]
X = [child, child, mother, father, cousin, father]
X = [child, child, father, father, cousin, father]
X = [child, father, husband, mother, cousin, cousin]
X = [child, husband, mother, father, child, mother]
X = [child, husband, mother, father, cousin, cousin]
X = [child, husband, mother, husband, mother, sibling]
X = [husband, mother, cousin, child, husband, mother]
X = [child, child, mother, father, cousin, father, sibling]
X = [child, child, mother, father, cousin, husband, mother]
X = [child, child, mother, husband, mother, cousin, father]
X = [child, child, father, father, cousin, father, sibling]
X = [child, child, father, father, cousin, husband, mother]
X = [child, child, father, husband, mother, cousin, father]
X = [child, child, husband, mother, father, cousin, father]
X = [child, husband, mother, husband, mother, cousin, cousin]
X = [child, child, mother, father, cousin, father, child, mother]
X = [child, child, mother, father, cousin, father, cousin, cousin]
X = [child, child, mother, father, cousin, husband, mother, sibling]
X = [child, child, mother, husband, mother, cousin, father, sibling]
X = [child, child, mother, husband, mother, cousin, husband, mother]
X = [child, child, father, father, cousin, father, child, mother]
X = [child, child, father, father, cousin, father, cousin, cousin]
X = [child, child, father, father, cousin, husband, mother, sibling]
X = [child, child, father, husband, mother, cousin, father, sibling]
X = [child, child, father, husband, mother, cousin, husband, mother]
X = [child, child, husband, mother, father, cousin, father, sibling]
X = [child, child, husband, mother, father, cousin, husband, mother]
X = [child, child, husband, mother, husband, mother, cousin, father]
X = [child, child, mother, father, cousin, husband, mother, cousin, cousin]
X = [child, child, mother, husband, mother, cousin, father, child, mother]
X = [child, child, mother, husband, mother, cousin, father, cousin, cousin]
X = [child, child, mother, husband, mother, cousin, husband, mother, sibling]
X = [child, child, father, father, cousin, husband, mother, cousin, cousin]
X = [child, child, father, husband, mother, cousin, father, child, mother]
X = [child, child, father, husband, mother, cousin, father, cousin, cousin]
X = [child, child, father, husband, mother, cousin, husband, mother, sibling]
X = [child, child, husband, mother, father, cousin, father, child, mother]
X = [child, child, husband, mother, father, cousin, father, cousin, cousin]
X = [child, child, husband, mother, father, cousin, husband, mother, sibling]
X = [child, child, husband, mother, husband, mother, cousin, father, sibling]
X = [child, child, husband, mother, husband, mother, cousin, husband, mother]
X = [child, child, mother, husband, mother, cousin, husband, mother, cousin, cousin]
X = [child, child, father, husband, mother, cousin, husband, mother, cousin, cousin]
X = [child, child, husband, mother, father, cousin, husband, mother, cousin, cousin]
X = [child, child, husband, mother, husband, mother, cousin, father, child, mother]
X = [child, child, husband, mother, husband, mother, cousin, father, cousin, cousin]
X = [child, child, husband, mother, husband, mother, cousin, husband, mother, sibling]
X = [child, child, husband, mother, husband, mother, cousin, husband, mother, cousin, cousin]
...
```
Видно, что цепочки иногда доходят до очень длинных. Проверим корректность работы программы и разберем какой-нибудь сложный на вид результат.
```prolog
?- related(X,'Борис Макаров','Валерий Макаров').
X = [child, husband, mother, husband, mother, cousin, husband, mother, cousin]
```
Пройдем данную цепочку вручную (для проверки):

Борис Макаров ->(ребенок)->Алексей Макаров->(муж)->Светлана Макарова ->(мать)->Антон Макаров->(муж)->Анастасия Акчурина->(мать)->Иван Макаров->(двоюродный брат)->Владимир Макаров->(муж)->Александра Мышляева->(мать)->Алиса Макарова->(двоюродный брат)->Валерий Макаров

Таким образом, мы убедились что программа работает корректно.

Однако, на этом же примере мы видим и главную проблему данной программы - поскольку из-за отсутствия предиката пола по условию, мы не можем знать пол ребенка, если у него нет детей, в моей программе были использованы общие предикаты child,sibling,cousin, которые, в целом, давали корректный результат, но значительно осложняли проверку какой-либо конкретной цепочки. Это можно решить, например, выводя на этапе поиска каждого человека, через которого проходит путь в текущей итерации. Тогда мы сможем увидеть имя человека, который считался общим предикатом child,sibling или cousin. Это можно просто реализовать, выводя параметр Chain в предикате запроса related ```... reverse(R, Chain), write(Chain), add(Chain, [], Relation1) ... ```. Однако, я не стал это делать, ввиду и так большого объема результатов. Но все же, я отдельно реализовал у себя данный вывод, и убедился в корректности работы алгоритма. Для рассмотренного выше примера, новый вывод выглядел был так:
```prolog
[Борис Макаров, Алексей Макаров, Светлана Макарова, Антон Макаров, Анастасия Акчурина, Иван Макаров, Владимир Макаров, Александра Мышляева, Алиса Макарова, Валерий Макаров]
X = [child, husband, mother, husband, mother, cousin, husband, mother, cousin]
```
## Определение степени родства - (второй вариант)
Изменив всего несколько строк, можно реализовать более подробные предикаты: вместо "ребенок" - "сын", "дочь", вместо "брат/сестра" - "брат", "сестра" (аналогично с двоюродными). В отчет я решил добавить решение согласно варианту, но дополнительно выкладываю решение с более подробными предикатами. Для этого можно либо добавить людям, не имеющим детей, предикат пола, либо сделать "фиктивными" родителями, поместив их на нужное место в предикате parents. Я выбрал вариант с предикатом пола. 

Добавочные факты:
```prolog
sex('Валерий Макаров',male).
sex('Сергей Макаров',male).
sex('Алиса Макарова',female).
sex('Иван Макаров',male).
sex('Олег Акчурин',male).
sex('Арсений Володьков'',male).
```
Новые отношений родства:
```prolog
% было - "relative(child, A, B) :- parents(A, _, B); parents(A, B, _)."    
% стало:
relative(son,A,B) :- child(A,B), (parents(_,A,_);sex(A,male)).
relative(daughter,A,B) :- child(A,B), (parents(_,_,A);sex(A,female)).

% было - "relative(sibling, A, B) :- sibling(A, B)."
% стало:
relative(brother, A,B) :- sibling(A,B), (parents(_,A,_);sex(A,male)).
relative(sister, A,B) :- sibling(A,B), (parents(_,_,A);sex(A,female)).

% было - relative(cousin, A, B) :- cousin(A, B).
% стало:
relative(cousin_brother, A,B) :- cousin(A, B), (parents(_,A,_);sex(A,male)).
relative(cousin_sister, A,B) :-  cousin(A, B), (parents(_,_,A);sex(A,female)).
```
Новый словарь родственников:
```prolog
relations(X):- member(X, [father, mother, brother, sister, husband, wife, cousin_brother, cousin_sister, son,daughter]).
```
Пример работы:
```prolog
?- related(R,'Валерий Макаров','Алиса Макарова').
R = [cousin_sister]
R = [daughter, brother, father]
R = [daughter, son, father, father]
R = [daughter, son, mother, father]
....
```
## Естественно-языковый интерфейс

Для реализации запросов на естественном языке, я использовал предикат, принимающий на вход список из слов, и выделяющий из него ключевые слова: имена и отношения. Затем, мы определяем тип вопроса, проверяем корректность основной грамматики и производим поиск ответа. Для поиска ключевых слов и проверки корректности вариативных частей вопроса, предикат сверяется со словарями.

Словари:
```prolog
relations(X):- member(X, [child, father, mother, sibling, husband, wife, cousin]).
pronouns(X) :- member(X,[he,she,it]).
pronouns_s(X) :- member(X, [his,her,its]).
verb_to_be(X):- member(X, [is,are]).
```
Всего в моей программе рассматривается 3 типа вопросов:

Кто является заданным родственником заданного человека - who is 'Name' 'Relation'?  - просто считываем параметр Name и подставляем его в предикат relative.

Является ли кто-то заданным родственником заданного человека - is 'Name1' 'Name2' 'Relation'? - аналогично первому вопросу, но считывается два разных имени и также подставляются в предикат relative.

Сколько заданных родственников у заданного человека - how many 'Relation' 'Name' have? - с помощью предиката findall ищем всех родственников через предикат relative, в ответе выводим длину полученного списка.

Рассмотрим пример предиката, определяющего тип вопроса:
```prolog
check_type_how(Q) :- 
    Q = [how,many|_],      % в начале вопроса обязательно идет грамматическая конструкция "how many"
    last(Q,?),             % в конце предложения ставится знак вопроса
    without_last(Q,Q1),    % (промежуточный предикат)
    last(Q1,have),         % слово, к которому ставится знак вопроса, обязательно должно быть "have"
    find_agent(Q,Name),    % убеждаемся, что в предложении присутствует только одно имя
    my_remove(Name,Q,Q2),  
    not(find_agent(Q2,_)),
    find_relation(Q,Rel),  % убеждаемся, что в предложении присутствует только одно название взаимоотношений
    my_remove(Rel,Q,Q3),
    not(find_relation(Q3,_)).
```    

Главный предикат, проверяет входные данные на тип вопроса, выделяет ключевые слова и ищет ответ. Таким образом, мы не обращаем внимание на сторонние слова (прилагательные, заключающие конструкции в некоторых вариантах), что будет продемонстрировано в результатах.
```prolog
q(Q) :- 
% how many Relation Name have ?
    (check_type_how(Q), find_agent(Q,Name),find_relation(Q,Rel),
        findall(Res,relative(Rel, Res, Name),L), length(L,X),
    	write(Name),write(" has "),write(X),write(" "),write(Rel),write("s");
% who is Name Relation ?     
    check_type_who(Q), find_agent(Q,Name),find_relation(Q,Rel),
        relative(Rel, Res, Name), write(Res), write(" is "), write(Name), write("'s "), write(Rel);
% is Name1 Name2 Relation ?    
   check_type_is(Q), find_relation(Q,Rel),find_agent(Q,Name1),my_remove(Name1,Q,Q1),find_agent(Q1,Name2),
        (relative(Rel, Name1, Name2) -> write("Yes. "),write(Name1),write(" is "),write(Name2),write(" "),write(Rel);
      	not(relative(Rel, Name1, Name2)) -> write("No. "),write(Name1),write(" is not "),write(Name2),write(" "),write(Rel));
% how many Relation Pronoun have?    
   check_type_how_obj(Q);
% who is Pronoun Relation?
   check_type_who_obj(Q);
% is Pronoun Name Relation?   
   check_type_is_obj_1(Q);
% is Name Pronoun Relation?
   check_type_is_obj_2(Q)).
```
Пример запроса:
```prolog
?- q([who,are,'Валерий Макаров',cousin,?]).
Сергей Макаров is Валерий Макаров's cousin
true
Алиса Макарова is Валерий Макаров's cousin
true

?- q([who,is,my,best,friend,"'s",'Валерий Макаров',father,?]).
Александр Макаров is Валерий Макаров's father

?- q([how,many,little,child,'Владимир Макаров',have,?]).
Владимир Макаров has 2 childs

?- q([is,'Владимир Макаров','Александр Макаров',sibling,?]).
Yes. Владимир Макаров is Александр Макаров sibling

?- q([are,'Владимир Макаров',and,'Александр Макаров',sibling,i,wonder,?]).
Yes. Владимир Макаров is Александр Макаров sibling
```
По данной программе можно сразу сделать несколько комментариев. 

Во-первых, из-за технических проблем не удалось реализовать тип вопросов, когда мы обращаемся к последнему упомянутому человеку (his,her). Это связано с тем, что самый удобный способ реализации такой возможности - операции nb_setval(), nb_getval() - были недоступны в моей среде разработки. Сама реализация довольно простая: с помощью nb_setval() мы запоминаем значение имени в вызванном предикате, и затем, если после него был вызван тип вопроса с his/her, то мы возвращаем через nb_getval() значение имени и работаем с ним. Несмотря на это, я реализовал проверку на тип вопроса с применением местоимений:
```prolog
?- check_type_is_obj_1([is,she,'Валерий Макаров',mother,?]).
true
?- check_type_who_obj([who,are,his,cousin,?]).
true
```
Таким образом, в главном предикате, после указанных проверок на тип вопроса, остается только скопировать соответствующие алгоритмы поиска ответа, только в процессе выделения имени, мы возьмем его не из предложения, а из функции nb_getval().

Во-вторых, можно было реализовать более подробную проверку грамматики, поскольку, программа не учитывает порядок некоторых элементов. Однако, по условию, мы задаем программе осмысленные вопросы, а значит, полагаем что порядок слов во входных данных будет грамматически корректным.

В-третьих, можно было реализовать некоторые слова из словарей во множественном числе. Например, многие названия отношений: sibling - siblings, child - children. Это можно было реализовать несложным предикатом, однако я решил не усложнять этим код.

## Выводы

 По сути, все 3 задачи курсового проекта представляли собой обработку некой базы данных, представленной в заданном виде. Благодаря курсовому проекту, я увидел основные преимущества языка prolog. То, что в императивных языках программирования заняло бы ощутимую часть кода, в прологе может реализовываться в одну или несколько строк. Особенно хорошо это было заментно при работе с данными в реляционном представлении. Из этого я сделал вывод, что декларативные языки, в том числе пролог, позволяют составлять короткие и понятные программы, способные обрабатывать гигантские базы данных и предоставлять результат в удобной для чтения форме. 
 
 Многие процессы в prolog автоматизированы, иначе говоря, prolog во время своей работы производит, в определенном смысле, поиск решений. Однако, иногда нам необходимо было выполнять поиск в специфических стуктурах, аналогичных графам. Я изучил такие алгоритмы поиска в графе как поиск в глубину (Depth First Search - DFS), поиск в ширину (Breadth First Search - BFS) и поиск с итерационным заглублением (Iterative deepening depth-first search - IDDFS). На языке Prolog удобно выполнять задачи поиска, поскольку в самой механике языка существует готовый процесс бэктрэкинга. Иначе говоря, если, например, при поиске в графе мы доходим до вершины-листа, из которой нет путей, пролог выполнит возвращение к предыдущим вершинам. По итогу я изучил основные алгоритмы поиска и на основе этих знаний выбрал оптимальный для конкретной задачи алгоритм поиска.

 Так же, prolog достаточно эффективен при решении задач анализа естественного языка, но только в случае, когда нам известны базовые правила данной грамматики. В этом случае, Prolog позволяет удобно и наглядно работать со словарями, выполнять поиск слова и подбирать форму этого слова, исходя из прописанных языковых правил.
 
 Таким образом, я познакомился с некоторыми механиками и возможностями языка prolog, увидел наглядно его преимущества и недостатки и выполнил поставленные в данном проекте задачи.

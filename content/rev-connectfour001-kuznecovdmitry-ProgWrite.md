https://github.com/ProgWrite/sConnectFour.git  
[Кузнецов Дмитрий]

Игра "Четыре в ряд", своеобразный гибрид игр "Крестики-Нолики" и "Тетрис". Программа выполнена в ООП стиле.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Без инструкции непонятно, как играть
```
   0    1    2    3    4    5    6    
0  .    .    .    .    .    .    .  
1  .    .    .    .    .    .    .  
2  .    .    .    .    .    .    .  
3  .    .    .    .    .    .    .  
4  .    .    .    .    .    .    .  
5  .    .    .    .    .    .    .  
6  .    .    .    .    .    .    .  
Красный игрок введите координаты (номер ряда и номер столбца без пробелов)
1 1
Введите 2 числа без пробелов
11
Правила игры не позволяют занять ячейку по данным координатам!
```

2. Неудачная игровая механика. В реальности отличительная особенность этой игры состоит в том, что игрок бросает фишку в выбранную им вертикальную линию и фишка под тяжестью своего веса падает вниз и занимает самую нижнюю свободную по горизонтами ячейку.
Поэтому в компьютерной игре игрок должен вводить не две координаты, а только одну- номер столбца. Дальше его фишка должна автоматически стать на свободную ячейку. 

## ХОРОШО

1. Доска с разноцветными фишками
```
   0    1    2    3    4    5    6    
0  .    .    .    .    .    .    .  
1  .    .    .    .    .    .    .  
2  .    .    .    .    .    .    .  
3 🔴    .    .    .    .    .    .  
4 🔴   🟡    .    .    .    .    .  
5 🔴   🔴   🟡    .    .    .    .  
6 🔴   🟡   🟡    .    .    .    .  
Красный игрок победил!
```

2. Используется ООП стиль

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Префикс "is" используется только в методах, которые возвращают boolean
```
String isInputValid() 
```

- Метод не проверяет ввод, он осуществляет ввод
```
String checkInput() {
  //...
  String input = SCANNER.nextLine();  
  //...
}
```

- Метод не проверяет победителя, он показывает результаты игры
```
void checkWinner()
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  


**2. Readme**

- Интересно, что в ридми в качестве размера стандартной доски указывается 8x8
```
Игра играется на квадратной доске 8x8... в стандартном варианте используется 8 на 8
```

Большинство сайтов не знает такого размера для этой игры. На видео в ютубе тоже не удлось найти игры на доске 8x8.  
Википедия:
```
Наиболее распространенный вариант, также называемый классическим, 7x6, а также 8x7, 9x7 и 10x7.
```

**3. Слишком длинные конструкции**, которые трудно понять.
```
return board.getCircle(new Coordinates(row, column)).getColor() == Color.RED ? GameState.RED_WINS : GameState.YELLOW_WINS;

//ЛУЧШЕ:
Color color = board.getCircle(new Coordinates(row, column)).getColor();
return color == Color.RED ? GameState.RED_WINS : GameState.YELLOW_WINS;
```
Вводи поясняющие переменные.

*Фаулер, "Рефакторинг", гл.6,п."Введение поясняющей переменной".*  


**4. Если нужно печатать текст** с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
return (" " + selectPictureForCircle(circle) + "  ");

//ПРАВИЛЬНО:
return " %s ".formatted(selectPictureForCircle(circle));
```

**5. enum Color**, цвет фишки

- Нарушение SRP и вообще сути енамов как таковых. Енам это просто перечисление, он не должен хранить какую-то прикладную логику. 
Тем более знать, враждует красный цвет с желтым или дружат- это ответственность более высокого порядка
```
public enum Color {
  RED,
  YELLOW;

  public Color opposite() {
    return this == RED ? YELLOW : RED;
  }
}

//ПРАВИЛЬНО:
public enum Color {
  RED,
  YELLOW
 }
```
Про енамы я писал тут: https://github.com/raketareview/simulation_review/blob/master/content/rev-sim056-yanda-xkodxdf.md п."Архитектура"

**6. abstract class Circle**, игровая фишка

- Крайне неудачное название, которое меньше всего подходит для обозначения игровой фишки. Вызывает ассоциации с геометрией. При реализации проектов используй терминологию текущей предметной области.  
Английская википедия:
```
 is a game in which the players choose a color and then take turns dropping colored tokens
```
Правильно- Token.

**7. RedCircle/YelowCircle extends Circle**

- Не нужно делать специальных классов для красной и желтой фишки, достаточно сделать базовый класс Circle не абстрактным и пользоваться им.

- Если класс заявлен красной фишкой, он не должен принимать цвет в консруктор, иначе возможны казусы 
```
public class Main {
  public static void main(String[] args) {
    RedCircle redCircle = new RedCircle(Color.YELLOW);
    System.out.println("redCircle color: " + redCircle.getColor());
  }
}

public class RedCircle extends Circle {
  public RedCircle(Color color) {
    super(color);
  }
}

//РЕЗУЛЬТАТ:
redCircle color: YELLOW
```

**8. record Coordinates**

+ (+)Старая добрая координата из Симуляции оказалась полезна и тут. Потому что не нарушает SRP
```
public record Coordinates(int row, int column) {
}
```

**9. Пакет board**

- Содержит два класса: Board и BoardConsoleRenderer. Но эти классы принципиально разные. Board это модель, а BoardConsoleRenderer это представление. Он должен находиться вместе с другими рендерарами(или в полном одиночестве) в пакете renderer.

**10. class Board**

По сути упрощенный аналог Карты из проекта "Симуляция". Все замечания, актуальные для карты симуляции, актуальны и для Board.

+ (+)Содержит только нужные методы, SRP не нарушает, это хорошо.

- При всех операциях с участием координат нужно проверять координату на корректность
```
public void setCircle(Coordinates coordinates, Circle circle) {
  circles.put(coordinates, circle);
}

public boolean isCellEmpty(Coordinates coordinates) {
  return !circles.containsKey(coordinates);
}
```

**11. class DetermineGameState**, определяет состояние игры: в игре, ничья, красный виграл, желтый виграл.

- Мне кажется, класс знает много лишнего. Сейчас он среди прочего знает что:
  - В игре играют два игрока, не больше, не меньше
  - Игроки эти имеют конкретные цвета- красный и желтый
```
return board.getCircle(new Coordinates(row, column)).getColor() == Color.RED ? GameState.RED_WINS : GameState.YELLOW_WINS;
```

Считаю это знание лишним для данного класса. Класс должен знать только то, что:
  - Есть доска, на которой находятся фишки разного цвета, неважно какого именно
  - Если одинаковые фишки одного цвета стоят в ряд, этот цвет выиграл
  - Если никто не выиграл и все клетки заняты, то ничья

Я бы это сдел примерно так
```
public class GameResultsAnalyzer {
  
  private final Board board;

  public GameResultsAnalyzer(Board board) {
    this.board = board;
  }

  private final boolean isDraw() {...}

  private final boolean isWin(Color color) {...}
}
```

- Магические цифры: 3.

**12. class Validation**

- Нарушение SRP, название класса не соответствует тому, что делает класс. Класс не только валидирует но и осуществляет ввод команд юзера.

- Метод делает несколько дел сразу: проверяет координату и пишет сообщение, если что-то не так
```
private boolean isCellAvailableForMove(Coordinates coordinates, Board board) {
  if (board.isCellEmpty(coordinates)) {
    return true;
  } else {
    System.out.println("На таких координатах уже располагается круг");
    return false;
  }
}

//ПРАВИЛЬНО:
private boolean isCellAvailableForMove(Coordinates coordinates, Board board) {
  return board.isCellEmpty(coordinates);
}
```
То же самое касается `boolean checkGameLogic(Coordinates coordinates, Board board)`.

**13. class BoardConsoleRenderer**

- Магическое: "   ", "    ".

**14. class Game**

- Дублирование
```
if (colorToMove == Color.RED) {
  System.out.println("Красный игрок введите координаты (номер ряда и номер столбца без пробелов)");
} else {
  System.out.println("Желтый игрок введите координаты (номер ряда и номер столбца без пробелов)");
}

//ПРАВИЛЬНО:
String colorName = toColorName(colorToMove);
System.out.printf("%s игрок введите координаты (номер ряда и номер столбца без пробелов) %n", colorName);
```

- В многопользовательских играх должен быть класс Игрок и передавать ход нужно между ними
```
private void printMessageForUser() {
  if (colorToMove == Color.RED) {
    System.out.println("Красный игрок введите координаты (номер ряда и номер столбца без пробелов)");
  } else {
    System.out.println("Желтый игрок введите координаты (номер ряда и номер столбца без пробелов)");
  }
}

private void setCircle(Coordinates coordinates) {
  if (colorToMove == Color.RED) {
    board.setCircle(coordinates, new RedCircle(Color.RED));
  } else {
    board.setCircle(coordinates, new YellowCircle(Color.YELLOW));
  }
}

//...
colorToMove = colorToMove.opposite();

//ПРАВИЛЬНО:
private void printTurnMessage() {
  Color color = currentPlayer.getCircle().getColor();
  String colorName = toColorName(color); 
  System.out.printf("%s игрок введите координаты (номер ряда и номер столбца без пробелов) %n", colorName);
}

private void setCircle(Coordinates coordinates) {
  Circle circle = currentPlayer.getCircle();
  board.setCircle(coordinates, circle);
}

private void changeCurrentPlayer() {
  currentPlayer = currentPlayer == firstPlayer ? secondPlayer : firstPlayer;
}
```

**15. class Main**, содержит точку входа main

+ (+)Только создает и запускает Game, это хорошо.

## ДЕКОМПОЗИЦИЯ

+ (+)Декомпозиция выполнена по правилам ООП.

- В многопользовательских играх должен быть класс Игрок, который используется для идентификации хода и хранит необходимые для совершения хода данные. Например в данном случае- свою игровую фишку
```
public class Player {
  private final Circle circle;

  public Player(Circle circle) {
    this.circle = circle;
  }

  public Circle getCircle() {
    return circle;
  }
}
```

Кроме прочего это позволяет делать ботов 
```
public abstract class Player {...}
public class RealPlayer extends Player {...}
public class Bot extends Player {...}
```

- Для игр, где игроки делают ходы одинаковыми фишками (4 в ряд, крестики-нолики, го etc) оптимальным является делать ходы не объектами, а енамами(да, енам это тоже объект, но ты понял).

- Есть много вариантов этой игры с разными цветами фишек, поэтому я бы сделал игровую фишку-енам тоже с разными цветами в количестве больше двух
```
public enum Token {
  RED,
  YELLOW,
  GREEN,
  WHITE
  //oth's
}
```

- Если можно будет делать разных игроков, то можно будет делать и разные игровые конфигурации
```
public class Main1 {
  public static void main(String[] args) {
    Player first = new RealPlayer(Token.RED);
    Player second = new RealPlayer(Token.YELLOW);
    Board board = new Board(7, 6);
    Game game = new Game(board, List.of(first, second));
    game.start();
  }
}

public class Main2 {
  public static void main(String[] args) {
    Player first = new RealPlayer(Token.GREEN);
    Player second = new Bot(Token.WHITE);
    Board board = new Board(7, 9);
    Game game = new Game(board, List.of(first, second));
    game.start();
  }
}
```

## ВЫВОД

Нужно тренировать навыйки нейминга. В целом неплохо.

n.1(128)
#ревью #четыревряд #connectfour
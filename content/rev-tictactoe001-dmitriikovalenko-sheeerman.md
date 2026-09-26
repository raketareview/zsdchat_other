https://github.com/sheeerman/tictactoe_OOP  
[Dmitrii Kovalenko]

Игра "Крестики-Нолики" в ООП стиле
```
Посмотрел теорию по ООП плюс три стрима с декомпозицией (крестики-нолики, змейку и шахматы) и стрим про процедурный стиль от Сергея. 
Решил для практики сделать рефакторинг крестиков-ноликов от Сергея в ООП стиле. 
Буду благодарен за ревью или замечания по коду, так как до этого с ООП не работал.
```

Я не буду делать полный разбор программы, например, ошибки нейминга или громоздкую логику определения состояния игры в `WinChecker`.

**Рассмотрю только применение ООП.** 

✅ public record Coordinate(int row, int col)

Идеальная координата для всех игр с прямоугольным полем.

❌️ enum CellState

- Какие-то цифровые коды, они не не описывают состояние ячейки.

И являются частью ответственности подсчёта выигрыша, а не ответственности идентификации значения в ячейке
```java
public enum CellState {
  EMPTY(0, ' '), X(1, 'X'), O(-1, 'O');
  //...
}

//ПРАВИЛЬНО:
public enum CellState {
  EMPTY(' '), X('X'), ZERO('0');
  //...
}
```

Это же енам, а не что-то бинарное.  
Так представь, что в этот енам добавили еще один символ, например для безумной игры втроём, то какой секретный код ты сюда добавишь?
```java
public enum CellState {
  EMPTY(0, ' '), X(1, 'X'), O(-1, 'O'), Y(<???>, 'Y');
  //...
}
``` 

- Нужно ли здесь хранить `EMPTY`- вопрос дискуссионный. 
Если бы класс назывался `Figure`, то однозначно нет. А так пусть будет.

✅ class Board

ОК.

👻 GameState checkGameState

Мне не нравится способ определения состояния игры. 
Метод знает слишком много правил общей игровой логики: играют только два игрока и заранее знает, как каждого из них зовут 
```java
public GameState checkGameState(Board board) 
```
В моем стриме другая логика определения результатов игры и я объяснял почему.  
Что-то типа такого:
```java
public boolean isWin(Board board, Figure figure) {...}
public boolean isDraw(Board board, List<Figure> figures) {...}
``` 
Но это не настолько принципиальный вопрос, так что пусть будет. 

✅ class Display

ОК.

👻  interface Player

На первый взгляд ничего не предвещает беды, но посмотрим, как метод `makeMove()` проявит себя в имплементациях 
```java
public interface Player {
  public Coordinate makeMove(Board board);

  CellState getSymbol();
}
```

❌️ class HumanPlayer implements Player

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель(а это модель) не должна ничего печатать в консоль
```java
public class HumanPlayer implements Player {

  private final Scanner scanner;
  private final Display display;
  private final CellState symbol;

  public HumanPlayer(Scanner scanner, Display display, CellState symbol) {
    this.scanner = scanner;
    this.display = display;
    this.symbol = symbol;
  }

  @Override
  public CellState getSymbol() {
    return symbol;
  }

  @Override
  public Coordinate makeMove(Board board) {
    //...
    String[] input = scanner.nextLine().split(" "); <-- ВВОД ДАННЫХ ИХ ПРЕДСТАВЛЕНИЯ
    //...
  }

  private boolean isValidMove(Coordinate coord, Board board) {
    //...
    display.printError("Неверный ввод. Введите два значения от 0-2 через пробел: ");  <-- ВЫВОД ДАННЫХ В ПРЕДСТАВЛЕНИЕ
    //...
    display.printError("Клетка заполнена. Выберите другую клетку: ");
  }
}
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах нужно будет менять код в этом классе, чтобы вывод осуществлялся по правилам этой среды, то есть, это лишняя причина для изменения класса.
В других средах(напр. Андроид) эту модель нельзя будет использовать- она там просто не скомпилируется. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Если попытаться сделать версию этой игры в `Swing` и использовать в ней классы-модели из этой консольной версии, то класс `HumanPlayer` придется переделывать.  
Следовательно, модель зависит от представления.

Кто-то подумает, что раз изначально не планируется кроме консольной версии делать версии для других сред ввода-вывода(Свинг, Андроид), то можно не особо париться с зависимостями моделей от представлений.  
Но дело не в том, будут другие версии программы или нет.  

Дело в том, что нарушение простых правил проектирования рано или поздно вылазит в совершенно неожиданных местах. 
Поэтому лучше их знать и не нарушать.

❌️ class BotPlayer implements Player 

- Нарушение SRP. То же, что в прошлом классе.

Этот класс может (и должен) иметь метод, кот который возвращает координату, куда бот хочет сделать ход.  
Но при этом бот не должен ничего печатать в консоль. 

И метод совершения хода он не должен наследовать от общего с `HumanPlayer` предка, этот метод должен появиться на уровне `BotPlayer`
```java
@Override
public Coordinate makeMove(Board board) {
  display.printBotPrompt();
  pause();
  return findRandomEmptyCell(board);
}

//ПРАВИЛЬНО:
public Coordinate makeMove(Board board) {
  return findRandomEmptyCell(board);
}
```

- Полиморфизм с `HumanPlayer`.

Да, на первый взгляд полиморфизм Бота с Реальным Плеером через общий интерфейс совершения хода выглядит гениальным и удобным.

Проблема в том, что тогда модель-плеер неизбежно начинает зависеть от представления.  
Потому что в конечном счете все равно как-то нужно получать ввод команд от юзера:  
Через кнопки клавиатуры в консоли, клик мышкой в свинге, нажатием пальца в андроиде
```java
public interface Player {
  public Coordinate makeMove(Board board);

  CellState getSymbol();
}

public class BotPlayer implements Player {
  //...

  @Override
  public Coordinate makeMove(Board board) {
    display.printBotPrompt();
    pause();
    return findRandomEmptyCell(board);
  }

}
public class HumanPlayer implements Player {
  //...

  @Override
  public Coordinate makeMove(Board board) {
    display.printCoordinatePrompt();    <- ВЫВОД ЮЗЕРУ

    do {
      String[] input = scanner.nextLine().split(" ");  <- ВВОД ОТ ЮЗЕРА
      int row = Integer.parseInt(input[0]);
      int col = Integer.parseInt(input[1]);
      Coordinate coord = new Coordinate(row, col);

      if (isValidMove(coord, board)) {
        return coord;
      }
    } while (true);
  }
}
```

Подробнее я это обговаривал в своем стриме.

- Нарушение SRP.

Пауза не относится к ответственности класса.  
Это ответственность общей игровой логики.  
Бот должен отвечать мгновенно. А через какие промежутки времени его будут опрашивать- не его забота.  

✅ class Game

ОК.

✅ class Main

ОК.

## ВЫВОД

В целом ок.

Стрим Сергея [Крестики-нолики в процедурном стиле](https://www.youtube.com/watch?v=PPikj1qHxrA)  
Мой стрим [Крестики-нолики в ООП стиле](https://t.me/zhukovsd_it_chat/53243/187097)

n.3(367)  
#ревью #tictactoe #ooptictactoe 
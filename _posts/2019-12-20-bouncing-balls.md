---
layout: post
categories: posts, tutorial
title: Мир прыгающих шаров. Это не то что вы думаете. Часть 1 [Перевод]
tags: [ tutorial, translate, Java, Swing, GameDev]
date-string: ДЕКАБРЬ 20, 2019
featured-image: 

---


# Введение
Данный цикл постов является переводом статьи из курса (https://www.ntu.edu.sg/home/ehchua/programming/java/J8a_GameIntro-BouncingBalls.html)[*"Java Game Programming"*]. Я наткнулся на эту статью в результате поисков в интернете о разработке игр в рамках фрейморка Swing. Ее объем и подробное изложение, казалось бы, такой простой темы как, имитация физики столкновений шариков в играх, меня заинтересовали, как надеюсь заинтересует и читателя. 

# Демо 

Добавлю позже 

# Случай 1. Шарик в коробочке

Рассмотрим самый простой способ имитации физики столкновений шарика о стенки прямоугольного контейнера. Это займет буквально несколько строк кода 

``` java
import java.awt.*;
import java.util.Formatter;
import javax.swing.*;
/**
 * Один отражающийся шарик в прямоугольной коробке.
 * Весь код в одном файле. Это неграмотный дизайн кода!
 */


// Расширяем поведение(добавляем) на основе базовой JPanel, для того что бы изменить отрисовку этого компонента 
public class BouncingBallSimple extends JPanel {
   // Зададим ширину и высоту контейнера для шарика
   private static final int BOX_WIDTH = 640;
   private static final int BOX_HEIGHT = 480;
  
   // Зададим характеристики шарика
   private float ballRadius = 200; // Радиус шарика
   // Координаты центра (x, y)
   private float ballX = ballRadius + 50; 
   private float ballY = ballRadius + 20; 
   private float ballSpeedX = 3;   // Скорость шарика по различным осям
   private float ballSpeedY = 2;
  
   private static final int UPDATE_RATE = 30; // Число отвечающее за количество обновлений экрана за единицу времени
  
   /** Создадим конструктор для создания UI и инициализации объектов */
   public BouncingBallSimple() {
      this.setPreferredSize(new Dimension(BOX_WIDTH, BOX_HEIGHT));
  
      // Дадим толчок нашему шарику (из отдельного потока)
      Thread gameThread = new Thread() {
         public void run() {
            while (true) { // вечный цикл обновления
               // Модифицируем координаты шарика по осям на некоторую дельту
               ballX += ballSpeedX;
               ballY += ballSpeedY;
               // Проверка на пересечение границ 
               // Если пересекли, то изменяем вектор скорости и координаты границ
               if (ballX - ballRadius < 0) {
                  ballSpeedX = -ballSpeedX; // Инвертация вектора движения (обычное отражение)
                  ballX = ballRadius; // Релокация шарика относительно границы
               } else if (ballX + ballRadius > BOX_WIDTH) {
                  ballSpeedX = -ballSpeedX;
                  ballX = BOX_WIDTH - ballRadius;
               }
               // Проверим так же для двух других 
               if (ballY - ballRadius < 0) {
                  ballSpeedY = -ballSpeedY;
                  ballY = ballRadius;
               } else if (ballY + ballRadius > BOX_HEIGHT) {
                  ballSpeedY = -ballSpeedY;
                  ballY = BOX_HEIGHT - ballRadius;
               }
               // вызываем перерисовку компонента
               repaint(); // Callback paintComponent()
               // Задержка между вызовами перерисовки компонента
               try {
                  Thread.sleep(1000 / UPDATE_RATE);  // миллисекунды
               } catch (InterruptedException ex) { }
            }
         }
      };
      gameThread.start();  // вызываем исполнение потока
   }
  
   /** Переопределяем поведение для отрисовки компонента */
   @Override
   public void paintComponent(Graphics g) {
      super.paintComponent(g);    // Вызываем базовую отрисовку компонента
  
      // Рисуем контейнер
      g.setColor(Color.BLACK);
      g.fillRect(0, 0, BOX_WIDTH, BOX_HEIGHT);
  
      // Рисуем шарик 
      g.setColor(Color.BLUE);
      g.fillOval((int) (ballX - ballRadius), (int) (ballY - ballRadius),
            (int)(2 * ballRadius), (int)(2 * ballRadius));
  
      // Выводим информацию о шарике 
      g.setColor(Color.WHITE);
      g.setFont(new Font("Courier New", Font.PLAIN, 12));
      StringBuilder sb = new StringBuilder();
      Formatter formatter = new Formatter(sb);
      formatter.format("Ball @(%3.0f,%3.0f) Speed=(%2.0f,%2.0f)", ballX, ballY,
            ballSpeedX, ballSpeedY);
      g.drawString(sb.toString(), 20, 30);
   }
  
   /** точка входа нашей программы */
   public static void main(String[] args) {
      // вызываем отрисовку через композитный менеджер в новом потоке
      javax.swing.SwingUtilities.invokeLater(new Runnable() {
         public void run() {
            // Задаем основное окно программы
            JFrame frame = new JFrame("A Bouncing Ball");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setContentPane(new BouncingBallSimple());
            frame.pack();
            frame.setVisible(true);
         }
      });
   }
}

```

Теперь можно рассмотреть наш код подробнее. 

- наш класс BouncingBallSimple расширяет базовый класс в части относящейся к отрисовке компонента (мы определили что дополнительно нужно нарисовать)
- в конструкторе мы задали графические компоненты интерфейса. Также указали, что хотим обновлять состояние нашего шарика из нового потока
- с каждым обновлением, мы будем сдвигать наш шарик по координатным осям на величину его скорости, так же проверяя не пересеклись (*collision*) ли коодинаты шарика с границами нашего контейнера. Если шарик пересекает границу, мы *реагируем* меняя шарику позицию и скорость. Например, шарик отразится по горизонтали если его координаты пересекут границу по оси X и отразиться по вертикали, если координаты шарика пересекут ограничения по оси Y.
- мы расширили стандартное поведение `paintComponent()` для того что бы отрисовать необходимые графические объекты. Используя `fillRect()` мы отрисовываем прямоугольник контейнера, `fillOval()` для отрисовки шарика, `drawString()` для работы с текстом на экране.
- в методе `main()`, мы создаем объект окна, в котором будем производить все дальнейшие действия. Для отрисовки мы определили пользователький класс наследованный от JFrame. Код графики запускается в специальном UI-потоке *Event Dispatcher Thread(EDT)*, вызываемый из `javax.swing.SwingUtilities.invokeLater()`, так рекомендуют разработчики Swing. 

Несмотря на то, что данная программа работает, с точки зрения дизайна кода она оставляет желать лучшего (с точки зрения переиспользования кода и расширяемости программы). Более того, алгоритмы проверки на пересечение границ и реакции шарика несколько сыроваты и не расширяемы. К тому же в нашей программе нет контроля скорости шарика. 


# Случай 2. Шарик в объектно-ориентированном мире

Давайте перепишем нашу программу с шариком и контейнером в объектно-ориентированном стиле. Начнем с класса контейнера.

## Контейнер

![Контейнер класс]({{ site.baseurl }}\images\2019-12-20\GameBall_BoxClass.png) 

Класс содержит следующие поля: 
 - `minX, minY, maxX, maxY`, представляющий размеры контейнера
 Стоит сказать, что координатные оси в Java (как и во всей ОС Windows) графические координаты инвертированы *по горизонтали*, с центром осей (0,0) в левом верхнем углу, более наглядно это выглядит так: 

 
![Координатные оси окна](images\2019-12-20\Window_coordinate.png)

Класс контейнера содержит следующие методы: 
- конструктор класса, определяющий основные координаты окна (x, y), высота, ширина, цвет. Безопаснее использовать такое определение для определения прямоугольника. 
- метод для привязки или отвязки границ
- метод отрисовки, отрисовывающий окно (само себя)  при помощи графического контектста

```java
import java.awt.*;
/**
 * Прямоугольный контейнер, содержащий шарик
 */
public class ContainerBox {
   int minX, maxX, minY, maxY;  // размеры контейнера (package access)
   private Color colorFilled;   // цвет фона контейнера (background)
   private Color colorBorder;   // цвет границ контейнера
   private static final Color DEFAULT_COLOR_FILLED = Color.BLACK;
   private static final Color DEFAULT_COLOR_BORDER = Color.YELLOW;
   
   /** Конструктор */
   public ContainerBox(int x, int y, int width, int height, Color colorFilled, Color colorBorder) {
      minX = x;
      minY = y;
      maxX = x + width - 1;
      maxY = y + height - 1;
      this.colorFilled = colorFilled;
      this.colorBorder = colorBorder;
   }
   
   /** Конструктор с цветами по-умолчанию */
   public ContainerBox(int x, int y, int width, int height) {
      this(x, y, width, height, DEFAULT_COLOR_FILLED, DEFAULT_COLOR_BORDER);
   }
   
   /** Задаем или изменяем размеры окна */
   public void set(int x, int y, int width, int height) {
      minX = x;
      minY = y;
      maxX = x + width - 1;
      maxY = y + height - 1;
   }

   /** Отрисовываем самого себя и плюс необходимую графику */
   public void draw(Graphics g) {
      g.setColor(colorFilled);
      g.fillRect(minX, minY, maxX - minX - 1, maxY - minY - 1);
      g.setColor(colorBorder);
      g.drawRect(minX, minY, maxX - minX - 1, maxY - minY - 1);
   }
}


```
## Класс Шарика

![Класс шарика](images\2019-12-20\GameBall_BallClass.png)

Класс шарика содержит следующие поля: 
- x,y, radius, color, которые представляют центр шарика, его радиус и цвет
- speedX, speedY, которые представляют его скорость по осям, измеряемая в пикселях за единицу отрисовки

Под капотом, все числа выражаются в виде чисел с плавающей точкой для обеспечения более *гладкого* рендеринга, особенно для тригонометрических операций. Числа будут преобразованы в целые значения пикселей (для большинства игр достаточно 32-битной точности float, double, думаю, будет уже слишком)



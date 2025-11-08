# Functional interfaces

Для работы с лямбда-выражениями нужны функциональные интерфейсы. Однако вместо того, чтобы самим определять свои
интерфейсы, мы можем взять уже говотые, которые есть в JDK. ОНи покрывают наиболее часто встречающиеся случае и могут
использоваться в различных ситуациях. Рассмотрим основные из этих интерфейсов:

- Predicate<T>
- Consumer<T>
- Function<T,R>
- BiFunction<T, U, R>
- Supplier<T>
- UnaryOperator<T>
- BinaryOperator<T>

---

## Predicate<T>

Поверяет соблюдение некоторого условия. Если оно соблюдается, то возвращается значение true.
В качестве параметра лямбда-выражение принимает объект типа T:

    public interface Predicate<T> {
        boolean test(T t);
    }
    
    public class Program {
        public static void main(String[] args) {
             
            Predicate<Integer> isPositive = x -> x > 0;
             
            System.out.println(isPositive.test(5)); // true
            System.out.println(isPositive.test(-7)); // false
        }
    }

---

## Consumer<T>

Выполняет некоторое действие над объектом типа T, при этом ничего не возвращая:

    public interface Consumer<T> {
        void accept(T t);
    }
    
    public class Program {
        public static void main(String[] args) {
             
            Consumer<Integer> printer = x-> System.out.printf("%d долларов \n", x);
            printer.accept(600); // 600 долларов
        }
    }

---

## Function<T,R>

Представляет функцию перехода от объекта типа T к объекту типа R:

    public interface Function<T, R> {
        R apply(T t);
    }
    
    public class Program {
        public static void main(String[] args) {
             
            Function<Integer, String> convert = x-> String.valueOf(x) + " долларов";
            System.out.println(convert.apply(5)); // 5 долларов
        }
    }

---

## BiFunction<T, U, R>

Представляет функцию с двумя параметрами типа T и U и результатом типа R:

    public interface Function<T, U, R> {
        R apply(T t, U u);
    }

    public class Program {
        public static void main(String[] args) {
              
            BiFunction<Integer, Integer, Integer> sum = (x, y) -> x + y;
              
            System.out.println(sum.apply(3, 5)); // 8
            System.out.println(sum.apply(10, -2)); // 8
        } 
    }

---

## Supplier<T>

Не принимает никаких аргументов, но должен возвращать объект типа T:

    public interface Supplier<T> {
        T get();
    }

    public class Program {
        public static void main(String[] args) {
              
            Supplier<Integer> randomValue = ()-> (int)(Math.random() * 10) + 1;
              
            System.out.println("Случайное значение");
            System.out.println(randomValue.get());  // реальные вычисления происходят тут
        } 
    }

---

## UnaryOperator<T>

Принимает в качестве параметра объект типа T, выполняет над ними операции и возвращает результат операций в виде объекта
типа T:

    public interface UnaryOperator<T> {
        T apply(T t);
    }

    public class Program {
        public static void main(String[] args) {
             
            UnaryOperator<Integer> square = x -> x*x;
            System.out.println(square.apply(5)); // 25
        }
    }

---

## BinaryOperator<T>

Принимает в качестве параметра два объекта типа T, выполняет над ними бинарную операцию и возвращает ее результат также
в виде объекта типа T:

    public interface BinaryOperator<T> {
        T apply(T t1, T t2);
    }

    public class Program {
        public static void main(String[] args) {
             
            BinaryOperator<Integer> multiply = (x, y) -> x*y;
             
            System.out.println(multiply.apply(3, 5)); // 15
            System.out.println(multiply.apply(10, -2)); // -20
        }
    }

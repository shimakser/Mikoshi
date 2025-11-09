# Fork Join Pool

Это фреймворк для работы с задачами, которые можно рекурсивно делить на более мелкие подзадачи и выполнять параллельно.

### Основные концепции
1. RecursiveTask: Используется для задач, которые возвращают результат.
2. RecursiveAction: Используется для задач, которые не возвращают результат.

### Как работает
1. Разбиение задач: Метод compute() определяет, нужно ли делить задачу на подзадачи или решать её напрямую.
2. Параллельное выполнение: Подзадачи отправляются на выполнение с помощью метода fork().
3. Сбор результатов: Метод join() используется для получения результатов параллельных подзадач.


    public class ForkJoinExample {
    
    static class SumTask extends RecursiveTask<Long> {
        private final long[] numbers;
        private final int start;
        private final int end;

        private static final int THRESHOLD = 1000;
        
                SumTask(long[] numbers, int start, int end) {
                    this.numbers = numbers;
                    this.start = start;
                    this.end = end;
                }
        
                @Override
                protected Long compute() {
                    if (end - start <= THRESHOLD) {
                        long sum = 0;
                        for (int i = start; i < end; i++) {
                            sum += numbers[i];
                        }
                        return sum;
                    } else {
                        int mid = (start + end) / 2;
                        SumTask leftTask = new SumTask(numbers, start, mid);
                        SumTask rightTask = new SumTask(numbers, mid, end);
                        leftTask.fork(); // Запускаем левую задачу параллельно
                        long rightResult = rightTask.compute();
                        long leftResult = leftTask.join(); // Ждем завершения левой задачи
                        return leftResult + rightResult;
                    }
                }
            }
        
            public static void main(String[] args) {
                long[] numbers = new long[10000];
                for (int i = 0; i < numbers.length; i++) {
                    numbers[i] = i;
                }
        
                ForkJoinPool pool = new ForkJoinPool();
                SumTask task = new SumTask(numbers, 0, numbers.length);
                long result = pool.invoke(task);
                System.out.println("Сумма: " + result);
            }
    }
# CountDownLatch, CyclicBarrier

---

## CountDownLatch

Замок с обратным отсчетом предоставляет возможность любому количеству потоков в блоке кода ожидать, пока не завершится
определенное количество операций, выполняющихся в других потоках, перед тем как они будут «отпущены», чтобы продолжить
свою деятельность.

- В конструктор CountDownLatch обязательно передается количество операций, которое должно быть выполнено, чтобы замок
  «отпустил» заблокированные потоки.
- Блокировка потоков снимается с помощью счётчика: любой действующий поток, при выполнении определенной операции
  уменьшает значение счётчика. Когда счётчик достигает 0, все ожидающие потоки разблокируются и продолжают выполняться.
- Только текущий поток, в котором возникла проблема, выбрасывает исключение.
- Не может быть повторно использован после того, как счетчик достигнет нуля.

### Пример

- Метод countDown уменьшает счетчик, сигнализируя о том, что произошло событие,
- Методы await ожидают, пока счетчик не достигнет нуля, т. е. все события завершатся.

  public class TestAndGo implements Runnable {
  private CountDownLatch cln;

        public TestAndGo(CountDownLatch cln) {
            this.cln = cln;
        }
    
        @Override
        public void run() {
            System.out.println("в процессе тестирования");
            countDownLatch.countDown();
            System.out.println("пошел домой");
        }
  }

  public class Main {
  public static void main(String[] args) throws InterruptedException {
  var cnl = new CountDownLatch(3);
  IntStream.range(0, 3).forEach((i) ->
  new Thread(new TestAndGo(cnl)).start());
  countDownLatch.await();
  }
  }

---

## CyclicBarrier

Является точкой синхронизации, в которой указанное количество параллельных потоков встречается и блокируется. Как только
все потоки прибыли, выполняется опционное действие (или не выполняется, если барьер был инициализирован без него), и,
после того, как оно выполнено, барьер ломается и ожидающие потоки «освобождаются». В конструктор барьера обязательно
передается количество сторон, которые должны «встретиться», и, опционально, действие, которое должно произойти, когда
стороны встретились, но перед тем когда они будут «отпущены».

- Можете использовать снова, даже после того, как он сломается.
- Является альтернативой метода join(), который «собирает» потоки только после того, как они выполнились.

### Пример

У нас даже два CyclicBarrier — они ставятся в тех местах, где поток должен приостановиться дождаться остальных
потоков для продолжения работы. У нас это после «оторвать» и после «намазать» — сначала все рабочие отрывают обои,
ждут друг друга, и только потом приступают к следующему этапу «намазать». Аналогично после «намазать» каждый
останавливается и ждет, когда остальные 9 закончат мазать, и после этого все продолжают.

    public class Worker implements Runnable {
        private final CyclicBarrier b1;
        private final CyclicBarrier b2;
    
        public Worker(CyclicBarrier b1, CyclicBarrier b2){
            this.b1=b1;
            this.b2=b2;
        }
    
        @Override
        public void run() {
            try {
                System.out.println("оторвать");
                b1.await();
                System.out.println("намазать");
                b2.await();
                System.out.println("приклеить");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }

    public class Main {
        public static void main(String[] args) throws InterruptedException {
            CyclicBarrier b1=new CyclicBarrier(3);
            CyclicBarrier b2=new CyclicBarrier(3);
            IntStream.range(0,3).forEach((i)->
                new Thread(new Worker(b1,b2)).start());
        }
    }
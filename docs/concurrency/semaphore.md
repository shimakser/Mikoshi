# Semaphore

Это синхронизатор, который ограничивает доступ к какому-то ресурсу — семафор со счётчиком.

Управляет набором разрешений. Каждый acquire()
блокируется, если необходимо, до того, как разрешение доступно, затем получает его. Каждый release() добавляет разрешение, потенциально освобождая блокирующий получатель (acquirer). Используется для защиты дорогих ресурсов, которые доступны в ограниченном количестве,
например подключение к базе данных в пуле.

### Пример

- TestSemaphore<T> используется, чтобы ограничить количество элементов, которые можно добавить в Set<T>, по сути реализуя ограниченный по размеру набор.
- bound — максимальное количество элементов, которое можно добавить в множество.
- Collections.synchronizedSet — обеспечивает потокобезопасный доступ к HashSet.
- sem.acquire() блокирует поток, если нет доступных разрешений. Иначе — разрешение уменьшается на 1.
- Пытаемся добавить элемент в set. Если элемент уже существует, add() вернёт false.
- Если не удалось добавить элемент (wasAdded == false), нужно вернуть разрешение обратно (sem.release()), потому что acquire() уже уменьшил счётчик, но ресурс не был использован.
- Если элемент удалён — разрешение возвращается, как бы освобождая место в «ограниченном» множестве.


    public class TestSemaphore<T> {
    
        private final Set<T> set;
        private final Semaphore sem;
    
        public TestSemaphore(int bound) {
            this.set = Collections.synchronizedSet(new HashSet<T>());
            sem = new Semaphore(bound);
        }
    
        public boolean add(T o) throws InterruptedException {
            sem.acquire();
            boolean wasAdded = false;
            try {
                wasAdded = set.add(o);
                return wasAdded;
            } finally {
                if (!wasAdded)
                    sem.release();
            }
        }
    
        public boolean remove(Object o) {
            boolean wasRemoved = set.remove(o);
            if (wasRemoved)
                sem.release();
            return wasRemoved;
        }
    }
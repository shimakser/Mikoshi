# Stream API

---

## Методы 

1. stream.parallel()
- Использует ForkJoinPool.commonPool, который по умолчанию использует N потоков, где N = количество доступных процессоров (обычно = Runtime.getRuntime().availableProcessors()).

2. reduce(...)
- Агрегирует элементы в одно значение.
  Integer sum = list.stream().reduce(0, Integer::sum);
  Optional<Integer> sum = list.stream().reduce(Integer::sum);

3. groupingBy(...)
- Группирует по ключу
  Map<String, Long> countByCountry =
  people.stream().collect(Collectors.groupingBy(Person::getCountry, Collectors.counting()));

4. partitioningBy(Predicate)
- Разделяет на две группы: true / false.
  Map<Boolean, List<Person>> partition =
  people.stream().collect(Collectors.partitioningBy(Person::isAdult));

---

## Lazy stream

В Java Stream API все стримы ленивые (lazy) — это фундаментальное свойство, влияющее на производительность и поведение кода.

### Пример

    Stream<String> stream = Stream.of("a", "b", "c")
        .filter(s -> {
            System.out.println("filter: " + s);
            return true;
        });
    
    System.out.println("Перед forEach");
    stream.forEach(System.out::println);

#### Вывод
    
    Перед forEach
    filter: a
    a
    filter: b
    b
    filter: c
    c

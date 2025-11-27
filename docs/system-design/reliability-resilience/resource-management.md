# Resource Management

---

## Тёплые (warm) connection pools

Заранее открытые и поддерживаемые соединения к внешним системам (DB, HTTP clients), которые переиспользуются для запросов.

_Почему важно:_
* при пиковом трафике создание TCP/TLS соединения дорого (handshake, TLS, auth);
* warm pools уменьшают latency и CPU overhead.
* позволяет делать autoscaling без холодного старта.

_Практика:_
* держать достаточный пул соединений (min size) + max size;
* регулярно health-check и keepalive, чтобы conexiones не были закрыты NAT/firewall;
* pre-warm при старте новых инстансов (делать test queries), особенно при autoscale in;
* tune socket options: TCP_NODELAY, TCP keepalive.

_Проблемы:_ слишком большой pool = лишние resource (file descriptors), слишком маленький = connection starvation.

---
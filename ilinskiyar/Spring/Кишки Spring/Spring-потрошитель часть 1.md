
Как идет создание bean-а в Spring:
* до init-метода (первая настройка BPP - postgProcessBeforeIntitialization)
![[Pasted image 20240124180054.png]]
* после init-метода (второй проход BPP - postProcessAfterIntitialization)
![[Pasted image 20240124180241.png]]
Пояснение: Получаем полностью настроенные объекты

------

### BeanPostProcessor

![[Pasted image 20240124013106.png]]
Пояснение: Нужен для того, чтобы внести изменения в бин, до того как он попадет в контейнер (т.е. например: применить аннотации и т.д.)

---

Зачем нужен BPP?

![[Pasted image 20240124175154.png]]


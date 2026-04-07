# BGP. Продолжение

## Цель:
Настроить BGP между автономными системами. Организовать доступность между офисами Москва и С.-Петербург

## Задание:
  1. Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
  2. Настроите eBGP между провайдерами Киторн и Ламас
  3. Настроите eBGP между Ламас и Триада
  4. Настроите eBGP между офисом С.-Петербург и провайдером Триада
  5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.


<br>

### Топология
<center><img src="bgp_extension.png" align="middle"></center>

<br>

### Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
Для того чтобы настроить eBGP между офисом Москва и двумя интернет провайдерами Киторн и Ламас необходимо сначала запустить процесс BGP на соответствующих устройствах (маршрутизаторы: R14, R15, R22, R21).

```
R14(config)#router bgp 1001
R14(config-router)#bgp log-neighbor-changes
R14(config-router)#neighbor 100.78.0.1 remote-as 101
R14(config-router)#exit

R15(config)#router bgp 1001
R15(config-router)#bgp log-neighbor-changes
R15(config-router)#neighbor 100.77.0.5 remote-as 301
R15(config-router)#exit

R22(config)#router bgp 101
R22(config-router)#bgp log-neighbor-changes
R22(config-router)#neighbor 100.0.0.1 remote-as 520
R22(config-router)#neighbor 100.77.0.1 remote-as 301
R22(config-router)#neighbor 100.78.0.2 remote-as 1001
R22(config-router)#exit

R21(config)#router bgp 301
R21(config-router)#bgp log-neighbor-changes
R21(config-router)#neighbor 100.0.0.5 remote-as 520
R21(config-router)#neighbor 100.77.0.2 remote-as 101
R21(config-router)#neighbor 100.77.0.6 remote-as 1001
R21(config-router)#exit
```
































<br>

Полные файлы изменений приведены [здесь](config/)

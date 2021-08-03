Другой задачей контроллера деплоймента является обеспечение обновления на новую версию. Если обновить версию образа в объекте деплоймента, то контроллер удалит старые поды и поднимет новые. Сам процесс обновления может происходить по-разному в зависимости от настроек стратегии обновления. 

В нашем манифесте используется стратегия обновления RollingUpdate. Давайте обновим версию в манифесте на v2. **Нажимаем на кнопку copy to editor.**


И **применяем манифест**

**Во второй вкладке** можем наблюдать за тем, как одновременно создаются и удаляются поды. Т.е. в любой момент времени есть поды, которые отвечают на запросы.

Чтобы откатить деплоймент, достаточно вернуть версию назад. **Нажимем на кноку copy to editor.**

И **применяем обновленный манифест**

Во **второй вкладке** можем наблюдать за тем, как одновременно создаются и удаляются поды, и деплоймент возвращается на место. Дождемся пока деплоймент полностью откатится.

Мы также можем обновить версию деплоймента и откатить его с помощью императивных команд kubectl. 

Для обновления на новую версию можно использовать команду kubectl set image, которая обновляет образ в шаблоне пода деплоймента.

**Запускаем команду.** 

Во **второй вкладке** можем наблюдать за тем, как одновременно создаются и удаляются поды

А чтобы откатить можно использовать императивную команду kubect rollout undo. 

**Запускаем команду**

Во **второй вкладке** можем наблюдать за тем, как одновременно создаются и удаляются поды

Теперь посмотрим, как работает стратегия Recreate

**Правим стратегию в манифесте**

**Обновляем версию образа**

**Применяем манифест**

**Во второй вкладке** можем наблюдать за тем, как одновременно сначала все поды находятся в статусе Terminating.

А после их завершения, создаются новые.

Теперь можем удалить *деплоймент*. **Запускаем команду**. И **во второй вкладке** можем наблюдать за тем, как все поды деплоймента удаляются. 

Итак мы с вами посмотрели как контроллер деплоймента выполняет свои задачи - следит за количеством подов и производит обновление на новую версию. А также познакомились с командами kubectl для работы с деплойментом. 
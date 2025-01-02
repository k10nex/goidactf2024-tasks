# African Incident Investigation
Описание:

    Во время выполнения Специальной Компьютерной Операции в Африке сотрудники SHKOLO_CTF_RU заглянули на огонек в серверную местной АЭС. Там им приглянулся один древний компьютер, некогда управляющий SCADA-системой. Казалось бы, что может случиться с изолированной машиной на старой винде?...

## Теги
`#forensics` `#windows` `#image`

## Организаторам
Это задание состоит из трёх частей, которые должны открываться последовательно. То есть, Pt. 1 доступна сразу, Pt. 2 становится доступной после решения Pt. 1, а Pt. 3 становится доступной после решения двух предыдущих частей.
Описание, данное выше, должно присутствовать в карточке задания каждой части для удобства участников.
В свою очередь описания каждой части отдельно (данные ниже) должны присутствовать только в своих карточках.

## Выдать участникам
[Файл к заданию (Google Drive)](https://drive.google.com/drive/folders/18Agoas4E5SDlyTp1E_DgF72_F1rwv-Bu)




# African Incident Investigation, Pt. 1
Сложность: `Легко`

Описание: 

    Хм... Похоже, система не запускается. Выясните, из-за какого испорченного файла это происходит.
    Формат флага: goidactf{filename.extension}
## Решение
Импортируем виртуальную машину в VirtualBox и запускаем её. Нас встречает ошибка загрузчика:

![Система не запускается](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/broken2000.png)

Ошибка говорит о том, что не удается найти ядро системы. Начинаем гуглить, как это исправить, и буквально первой ссылкой в поисковой выдаче видим, *что чаще всего такое происходит из-за испорченного файла boot.ini :*

![Гуглим ошибку](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/google.png)

Чтобы в этом убедиться, откроем виртуальный диск машины в FTK Imager. Идем в корень диска и там находим boot.ini, который выглядит как-то подозрительно... 

![Странный boot.ini](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/strangebootini.png)

## Флаг

    goidactf{boot.ini}

# African Incident Investigation, Pt. 2
Сложность: `Средне`

Описание:

    Каким исполняемым файлом воспользовался злоумышленник для повышения привилегий?
    Формат флага: goidactf{filename.extension}

## Решение
*На самом деле для того, чтобы решить это задание, мы можем просто экспортировать логи системы из образа и начать копаться в них (автор вспомнил об этом только после бутылки шампанского в новогоднюю ночь, и такое бывает). Здесь будет рассмотрено решение, которое предполагалось изначально.*

Так как система не запускается из-за сломанного boot.ini, восстановим его. Для этого найдём чистый boot.ini, благо [гуглится](https://github.com/MicrosoftDocs/SupportArticles-docs/blob/main/support/windows-client/setup-upgrade-and-drivers/edit-boot-ini-file-windows-2000.md) он без проблем:

![Чистый boot.ini](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/cleanbootini.png)

Теперь нужно как-то отредактировать этот самый boot.ini на виртуальном диске машины. Так как FTK Imager не позволяет нам редактировать содержимое образа, добавим его внутрь другой виртуальной машины и отредактируем там.

![Добавляем диск в другую ВМ](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/anothervm.png)

При попытке отредактировать открыть файл внутри другой системы, нас встретит такое окно:

![Окно ошибки](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/insufficientpermns.png)

Чтобы долго не мучаться с разрешениями и перезаписью, просто сносим этот файл и создаем новый boot.ini, в который вставляем строки чистого файла, которые мы нагуглили.
Сохраняем изменения, выключаем виртуалку, убираем из нее жесткий диск изначальной машины и запускаем систему.

Нас встречает окно входа с заранее введенным пользователем scada-pc. На учетке нет пароля (в этом убеждаемся на рандом тыкнув enter), поэтому логинимся и идем искать следы атаки в логах.
Но здесь нас ждет облом - похоже, что у нас недостаточно прав для просмотра логов безопасности:

![Пытаемся посмотреть логи](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/scada-insufficientperms.png)

Похоже, придется лезть в админскую учетку. Её данные можно узнать через одну недоработку Win2000, а именно - попытаться зайти в оснастку "Users and Passwords", где мы увидим заранее введенное имя админской учетки, даже если это имя никогда в это окно не вводилось:

![Достали имя админки](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/gotadminlogin.png)

Админская учетка запаролена - это очевидно даже без проверки. Гуглим способ сброса пароля и натыкаемся на [видео индуса](https://www.youtube.com/watch?v=VWkRgck4iL0) (куда ж без них) и [ссылку в описании](https://pogostick.net/~pnh/ntpasswd/), которая ведет нас на сайт чудо-утилиты, сбрасывающей пароль на любых NT-based системах.

![Сайт тулзы](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/pwdrecoverysoft.png)

Скачиваем CD-версию, подключаем ее в нашу виртуалку и грузимся в тулзу. CLI-интерфейс софта оказывается крайне дружелюбным, и спустя буквально 3-4 нажатия кнопки enter мы видим заветный "Password cleared!":

![Убрали пароль](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/passcleared.png)

Дальше нужно не забыть нажать Y там, где нужно, и после надписи "Edit Complete" можно грузиться обратно в систему.

![Edit Complete](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/editcomplete.png)

Входим в учетку Administrator без пароля и отправляемся смотреть логи. Идем снизу вверх восстанавливая чейн, и в один момент заметим кое-что интересное:

![Killchain](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/logs1.png)

В 4:18:21 пользователем scada-pc из-под командной строки вызван **runas.exe**,  после происходит успешная аутентификация с использованием данных администратора в явном виде (злоумышленник, по всей видимости имел физический доступ к системе и знал пароль администратора), после чего уже от имени учетки Administrator выполняется команда **at**:

![at](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/African%20Incident%20Investigation/pics-for-writeup/logs2.png)

## Флаг

    goidactf{runas.exe}

# African Incident Investigation, Pt. 3
Сложность: `Легко`

Описание:

    Назовите полный номер тактики MITRE, которая была использована злоумышленником последней в хронолигическом порядке.
    Формат флага: goidactf{Txxxx.xxx}

## Решение
В нашем чейне последней исполняемой командой является **at**. At - один из способов управления запланированными задачами в Windows, значит первая часть искомой тактики - T1053 [(ссылка)](https://attack.mitre.org/techniques/T1053/).
На странице тактики в правой части расположены ссылки на подтехники. Одной их них является T1053.002 [(ссылка)](https://attack.mitre.org/techniques/T1053/002/), которая говорит об использовании именно утилиты **at.**

## Флаг

    goidactf{T1053.002}


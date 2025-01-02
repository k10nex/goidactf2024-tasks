# be crfl wth pictrs
Сложность: `Легко`

Описание:

`Наш слоник явно чем-то напуган... Разберись, в чём дело.`

## Теги
`#forensics` `#steganography` `#beginner`
## Выдать участникам
[task.png](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/be%20crfl%20wth%20pictrs/Task.png)
## Решение
Перед нами картинка, а значит первое, что мы расчехляем, это strings и binwalk.
Строки нам ничего не дают, но бинволк спасает - находим зашифрованный архив внутри нашей картинки:

![Архив внутри картинки](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/be%20crfl%20wth%20pictrs/pics-for-writeup/pic1.png)

Дальше два исхода - либо брутим, либо приглядываемся повнимательнее к исходной картинке (а точнее к одному слову на ней - `powershell`).

> Пароль от архива: `powershell`

Внутри архива нас ожидает еще одна картинка с забавным названием. Возвращаемся к нашим верным друзьям - и на этот раз пригождается strings, которым мы находим ни что иное, как powershell-однострочник:

![Однострочник](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/be%20crfl%20wth%20pictrs/pics-for-writeup/pic2.png)


> Однострочник: `sal a New-Object;Add-Type -A System.Drawing;$g=a
> System.Drawing.Bitmap("full\path\to\ithinkushouldntopenthis.png");$o=a
> Byte[] 768;(0..15)|%{foreach($x
> in(0..15)){$p=$g.GetPixel($x,$_);$o[($_*16+$x)*3]=$p.B;$o[($_*16+$x)*3+1]=$p.G;$o[($_*16+$x)*3+2]=$p.R}};$g.Dispose();IEX([System.Text.Encoding]::ASCII.GetString($o[0..370]))`

Изменяем путь к распакованной картинке внутри скрипта и запускаем его. Нужно подождать, и тогда нашему глазу предстанет полотно ошибок и какой-то ExceptionID внизу, крайне похожий на base64-закодированную строку:

![Вывод скрипта](https://github.com/k10nex/goidactf2024-tasks/blob/main/Forensics/be%20crfl%20wth%20pictrs/pics-for-writeup/pic3.png)


> Base64-строка: Z29pZGFjdGZ7N2gxNV9rMWRfa24wdzVfczBtMzdoMW42fQ

Раскодируем бейсу и получаем флаг.
## Флаг
    goidactf{7h15_k1d_kn0w5_s0m37h1n6}

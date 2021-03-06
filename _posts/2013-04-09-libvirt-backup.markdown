---
layout: post
title: "Бэкап виртуальных машин в libvirt с удалённого сервера"
date: '2013-04-09 03:06:00'
tags:
- kvm
- libvirt
- qemu
- ssh
- backups
- rsync
- vm
---

На работе приходится бэкапить около 10 виртуальных машин, усилий потратил на этот скрипт достаточно много, так что, думаю, стоит поделиться. Раньше использовал rdiff-backup, но он оказался излишне сложен и имел несколько неявных косяков, из-за которых я рисковал оказаться без резервных копий, в случае полного коллапса.

После этого пришлось потратить 40 минут на то, чтобы переписать всё на использование rsync, да и просто немного подефакторить написанный год назад скрипт.

Итак, [код](https://github.com/hordecore/useful_scripts/blob/master/virsync.sh)

Сейчас подтягивание изменений в ~100гб образов дисков занимает около полутора часов.

## Суть его работы

1. Получаем список виртуальных машин с флагом автостарт (симлинки на их XML'ки хранятся в директории /etc/libvirt/qemu/autostart/)
2. Паузим виртуалку, сохраняя её память в /tmp/
3. Получаем список дисков виртуалки из вывода `virsh dumpxml $vm`
4. Подтягиваем их rsync'ом на свою машину.
5. Восстанавливаем состояние виртуалки из /tmp
6. Переходим к следующей

Алгоритм вроде как максимально прост, не вредит самим виртуальным машинам. Запускается это в 3 утра, когда все сервисы, располагаемые в виртуальных машинах практически никому не нужны. По крону, раз в неделю и раз в месяц, дневные бэкапы копируются в недельные и в месячные соответственно.

## 6 лет спустя

Лол, а сейчас у нас целая облачная инфраструктура своя, которая автоматом всё это делает и даже задумываться не надо. Вдобавок у нас есть свой rsync, идеально подходящий для синхронизации дозаписи в конец файлов.

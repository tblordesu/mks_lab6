;***************************************************************
; Filename: millis_timer1_int1.asm
; Date: 02.05.2025
; Author: N.V.Vasilets
; Description: Timer 1 based millisecond timer with LCD and INT1
;***************************************************************

rs      equ P1.1
rw      equ P1.0
e       equ P1.3

dmks    equ 30h      ; десятки микросекунд (0–9)
hmks    equ 31h      ; сотни микросекунд (0–9)
ms      equ 32h      ; миллисекунды (0+)
started bit 0A0h.0   ; Флаг: таймер запущен или нет

;***************************************************************
; Векторы прерываний
;***************************************************************
org 0h
ajmp START

org 13h               ; INT1
ajmp int_1

org 1Bh               ; Timer 1 interrupt vector
ajmp timer1_isr

;***************************************************************
; Основная программа
;***************************************************************
org 100h
START:
    setb IT1
    setb EX1
    setb EA

    ; Инициализация ЖКИ (4-бит, младшие биты)
    clr rs
    clr rw

    ; Последовательность инициализации (4-bit)
    mov P2, #02h
    setb e
    clr e
    lcall delay

    mov P2, #02h
    setb e
    clr e
    lcall delay

    mov P2, #08h
    setb e
    clr e
    lcall delay

    mov P2, #00h
    setb e
    clr e
    lcall delay

    mov P2, #0Ch       ; Вкл дисплей, выкл курсор
    setb e
    clr e
    lcall delay

    clr rs
    mov P2, #08h       ; Установка курсора на начало
    setb e
    clr e
    lcall delay

    mov P2, #00h
    setb e
    clr e
    lcall delay

    ; Переключение в режим данных
    setb rs

    ; Вывод строки ФИО
    mov dptr, #0FD0h
str1_char:
    clr A
    movc A, @A+dptr
    jz forever
    mov R0, A
    ; старшие 4 бита
    mov A, R0
    anl A, #0F0h
    swap A
    clr A.4
    clr A.5
    clr A.6
    clr A.7
    mov P2, A
    setb e
    clr e
    lcall delay
    ; младшие 4 бита
    mov A, R0
    anl A, #0Fh
    mov P2, A
    setb e
    clr e
    lcall delay
    inc dptr
    sjmp str1_char

forever:
    sjmp forever

;***************************************************************
; Обработчик INT1: Старт / Стоп
;***************************************************************
int_1:
    jb started, stop_timer

start_timer:
    ; Настройка таймера 1 (10 мкс)
    mov TMOD, #10h         ; таймер 1 в режиме 16 бит
    mov TH1, #0FFh
    mov TL1, #0F6h         ; 65536 - 10 = 0xFFF6

    mov dmks, #0
    mov hmks, #0
    mov ms, #0

    setb ET1
    setb TR1

    ; Очистка строки
    clr rs
    mov P2, #0Ch
    setb e
    clr e
    lcall delay
    mov P2, #00h
    setb e
    clr e
    lcall delay

    clr rs
    mov P2, #00h
    setb e
    clr e
    lcall delay
    mov P2, #0Fh
    setb e
    clr e
    lcall delay

    setb started
    reti

stop_timer:
    clr TR1
    clr ET1
    clr started

    ; Переход на вторую строку (0xC0 = 11000000b)
    clr rs
    mov P2, #0Ch
    setb e
    clr e
    lcall delay
    mov P2, #00h
    setb e
    clr e
    lcall delay

    setb rs

    ; Вывод миллисекунд
    mov A, ms
    add A, #'0'
    lcall lcd_putc

    ; Точка
    mov A, #'.'
    lcall lcd_putc

    ; Сотни микросекунд
    mov A, hmks
    add A, #'0'
    lcall lcd_putc

    ; Десятки микросекунд
    mov A, dmks
    add A, #'0'
    lcall lcd_putc

    ; m
    mov A, #'m'
    lcall lcd_putc

    ; s
    mov A, #'s'
    lcall lcd_putc

    reti

;***************************************************************
; Прерывание таймера 1
;***************************************************************
timer1_isr:
    clr TF1
    mov TH1, #0FFh
    mov TL1, #0F6h
    inc dmks
    mov A, dmks
    cjne A, #10, t1_exit
    mov dmks, #0
    inc hmks
    mov A, hmks
    cjne A, #10, t1_exit
    mov hmks, #0
    inc ms  
    mov A, ms 
    cjne A, #10, t1_exit 
    mov ms, #0 
t1_exit:
    reti

;***************************************************************
; LCD putc для 4-битной передачи через P2.0–P2.3
;***************************************************************
lcd_putc:
    mov R0, A
    mov A, R0
    anl A, #0F0h
    swap A
    mov P2, A
    setb e
    clr e
    lcall delay

    mov A, R0
    anl A, #0Fh
    mov P2, A
    setb e
    clr e
    lcall delay
    ret

;***************************************************************
; Задержка
;***************************************************************
delay:
    nop
    nop
    nop
    nop
    nop
    ret

;***************************************************************
; ФИО (первая строка)
;***************************************************************
org 0FD0h
    db 'Bulatov A.Yu. 4241', 0

end

# 4_BGT
## Reikalingi instaliavimai
 + Truffle IDE (Node ir npm tinkamas versijas jau turiu)
 + Meta Mask Google Chrome extention
 + Ganache

## 1. Verslo modelis

B2C meno įrankių parduotuvė. B2C (bussines to customer) - Verslas parduoda prekes vartotojui, o nuosavybės teisė perduodama po apmokėjimo.
Šiame verslo modelyje dalyvauja:
 + Pardavėjas – parduoda meno įrankius (pieštukus, dažus, drobes ir kt.)
 + Pirkėjas – perka meno įrankius, naudodamasis decentralizuota aplikacija
 + Kurjeris – pristato prekes pirkėjui

Išmaniosios sutarties funkcijos:
 + Automatizuoja užsakymo vykdymą ir mokėjimo procesą
 + Užtikrina, kad prekės bus pristatytos tik gavus apmokėjimą
 + Sekama užsakymo būsena (gamyba, išsiuntimas, pristatymas)
 + Lėšos pervedamos pardavėjui tik po sėkmingo pristatymo.

Logika:
 + Užsakymo pateikimas:
   Pirkėjas pasirenka prekes ir pateikia užsakymą per decentralizuotą aplikaciją.
   Pirkėjas atlieka mokėjimą, o pinigai laikomi išmaniojoje sutartyje kaip užstatas.
   
 + Užsakymo vykdymas:
   Pardavėjas gauna užsakymo informaciją ir pradeda vykdymą.
   Prekės pereina į „Gamybos“ būseną, o vėliau į „Paruošta siuntimui“ būseną.
   
 + Siuntimas:
   Pardavėjas perduoda prekes kurjeriui, o sutartis automatiškai atnaujina užsakymo būseną į „Išsiųsta“.
   
 + Pristatymas:
   Kurjeris pristato prekes pirkėjui ir pažymi užsakymą kaip „Pristatyta“ per decentralizuotą aplikaciją.
   
 + Apmokėjimas pardavėjui:
   Kai kurjeris patvirtina, kad užsakymas pristatytas, išmanioji sutartis automatiškai perveda prekių kainą pardavėjui.
   
 + Ginčų sprendimas (jei reikalinga):
   Jei pirkėjas nepatenkintas pristatymu (pvz., prekės sugadintos), jis gali inicijuoti ginčo procesą.
   Ginčo metu lėšos lieka užrakintos, kol pasiekiamas sprendimas.

   
## 2. Verslo logikos realizacija

## 3. Testavimas

## 4. Logų peržiūra

## 5. Front-end

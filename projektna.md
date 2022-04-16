# Analiza nepremičnin in prihodkov prebivalstva

Sodelujoči:

- Rok Švikart
- Martin Čučkin
- Luka Kalin

# Povezave do podatkov:

Podatki o nepremičninah: https://podatki.gov.si/dataset/surs0419030s, 
                                          https://podatki.gov.si/dataset/evidenca-trga-nepremicnin
                                          
Podatki o prihodkih: https://www.gov.si/teme/minimalna-placa/, 
                       https://pxweb.stat.si/SiStatData/pxweb/sl/Data/-/0701011S.px


# Uvod

Nepremičninski trg je zelo pomemben za vsakega posameznika, saj vsi potrebujemo prebivališče ali delovne prostore, če želimo ustanoviti svoje podjetje ali manjši posel. Zaradi gotovosti, da se bomo v prihodnosti vsi srečevali s tem trgom, smo se v skupini odločili narediti projekt o nepremičninskem trgu in o njegovi možni prihodnosti. 

# Opis problema

Pri nalogi si želimo predstaviti obnašanje nepremičnin na trgu, ter prihodkov prebivalstva. Želimo analizirati trende o cenah, ali rastejo, padajao ali pa stagnirajo. S to analizo podatkov, bi lahko odgovorili na vprašljivo prihodnost nepremičninskega trga in vprašanja ali bo oseba glede na povprečne prihodke v Sloveniji na tem trgu lahko sodelovala, ali je to v današnjih časih vedno težje.

# Podatki

Pri vmesenm poročili smo uporabili tri različne podatkovne datoteke, in sicer:

    - min_place.csv - manjša datoteka z minimalnimi plačami v Sloveniji skozi leta 2010 - 2021
    
    - povp_place.csv - datoteka, ki vsebuje povprečno bruto plačo za mesec in tromesečje in povprečno neto plačo za posamezen mesec in tromesečje v obdobjih 2014 - 2021

    - st_in_vrednost_nepremicnin.csv - datoteka, ki vsebuje podatke o številu transakcij in njihovih skupnih prihodkih glede na četrtletja v obdobju 2010 - 2021 za posamezne tipe nepremičnine, ki pa so:
        - Nova stanovanja
        - Nove družinske hiše
        - Rabljena stanovanja
        - Rabljene družinske hiše
    
Preostanejo še podatki o evidenci trga nerpemičnin, katerih se do sedaj še nismo lotili in ostajajo za nadaljevanje projektne naloge. S temi podatki želimo predstaviti obnašanje tudi glede na regije Slovenije, saj ima datoteka za vsako transakcijo, tudi podatek o občini, katere lahko nato razdelimo po regijah.

# Knjižnice



```python
import csv
from csv import DictReader
import re
import matplotlib
from matplotlib.pyplot import figure
import matplotlib.pyplot as plt
import numpy as np
import sys
```

# Branje podatkov

Podatke smo brali s pomočjo DictReader-ja.

Odpreta datoteka o minimalnih plačah, vstavljena v dictionary.


```python
reader = DictReader(open('podatki/min_place.csv', 'rt', encoding='utf-8'))

leto_min_placa = dict()

for row in reader:
    leto_min_placa[row["Leto"]] = row["Placa"]

print(leto_min_placa)
```

    {'2010': '734.15', '2011': '748.10', '2012': '763.06', '2013': '783.66', '2014': '789.15', '2015': '790.73', '2016': '790.73', '2017': '804.96', '2018': '842.79', '2019': '886.63', '2020': '940.58', '2021': '1024.24'}
    

Branje datoteke st_in_vrednost_nepremicnin.csv, ter risanje grafa o obnašanju števila transakcij te rnjihovih skupnih vrednosti.


```python
replacer = {'\"': '', '\n': ''}
matrika = np.empty([1,95])
with open("podatki/st_in_vrednost_nepremicnin.csv", "r") as file_st_in_vrednost:
    for x in file_st_in_vrednost:
        for key, value in replacer.items():
            vrstica = [s.replace(key, value) for s in x.split(",")]
        matrika = np.vstack((matrika,vrstica))
matrika = matrika[1::]
del x
xVrednosti = matrika[:1:]
xVrednosti = xVrednosti.ravel()
xVrednosti = xVrednosti[1::]
xVrednosti = xVrednosti[::2]
pom = ["a"]
for vrednost in xVrednosti:
    pom = np.vstack((pom,vrednost.split(" ")[0]))
xVrednosti = pom[1::]
xVrednosti = xVrednosti.ravel()

xVrednosti = [x.replace('"','') for x in xVrednosti]
xVrednosti = [x if x.endswith("Q1") else f"{x[2:4]}{x[4:]}" for x in xVrednosti ]

stare = matrika[2:3:]
nove = matrika[5:6:]

stare = stare.ravel()
nove = nove.ravel()

stare = stare[1::]
nove = nove[1::]

stare = stare.astype(int)
nove = nove.astype(int)

stareTransakcije = stare[::2]
noveTransakcije = nove[::2]
stareVrednost = stare[1::2]
noveVrednost = nove[1::2]


```

Branje datoteke povp_place.csv.


```python
reader = DictReader(open('podatki/povp_place.csv', 'rt'))

meseci, bruto, bruto3, neto, neto3 = [],[],[],[],[]



for row in reader:
    meseci.append(row["MESEC"])
    bruto.append(row["Bruto placa Placa za mesec [EUR]"])
    bruto3.append(row["Bruto placa Placa za tromesecje [EUR]"])
    neto.append(row["Neto placa Placa za mesec [EUR]"])
    neto3.append(row["Neto placa Placa za tromesecje [EUR]"])

povp_place = np.array([meseci,bruto,bruto3,neto,neto3])

```


```python
#!!!!!!PAZI TABELA IMA VREDNOSTI V STRING FORMATU, NAREDI PRETVORBO ČE JO RABIŠ!!!!!!!!!!

#primer uporabe... stolpci tabele : meseci, bruto, povp zadnjih 3 bruto, neto, povp zadnjih 3 neto
# for placa in povp_place[1,:3]: #bruto place za prve 3 vnose
#     print(placa)

# #koliko je placa specificen datum
# msk = povp_place[0] == "2014M08"
# povp_place[:,msk] #izpisi vse
```

Priprava regij za prevrjanje v nadaljnem delu. Nameravamo pregledovati nepremičninski trg v posameznih regijah, zato smo pripravili set-e o občinah, ki so v regijah, za lažje določanje pripadanja podatkov ustreznim regijam.


```python
#Podatki: katera občina pripada kateri regiji
regije = {
    "obalnokraska" : set(("ANKARAN", "DIVAČA", "HRPELJE-KOZINA", "IZOLA", "KOMEN", "KOPER", "PIRAN","SEŽANA")),
    "primorskonotranjska" : set(("BLOKE", "CERKNICA", "ILIRSKA BISTRICA", "LOŠKA DOLINA", "PIVKA", "POSTOJNA")),
    "goriska" : set(("AJDOVŠČINA", "BOVEC", "BRDA", "CERKNO", "IDRIJA", "KANAL", "KOBARID", "MIREN-KOSTANJEVICA", "NOVA GORICA", "RENČE-VOGRSKO", "ŠEMPETER-VRTOJBA", "TOLMIN", "VIPAVA")),
    "gorenjska" : set(("BLED", "BOHINJ", "CERKLJE NA GORENJSKEM", "GORENJA VAS-POLJANE", "GORJE", "JESENICE", "JEZERSKO", "KRANJ", "KRANJSKA GORA", "NAKLO", "PREDDVOR", "RADOVLJICA", "ŠENČUR", "ŠKOFJA LOKA", "TRŽIČ", "ŽELEZNIKI", "ŽIRI", "ŽIROVNICA")),
    "osrednjeslovenska" : set(("BOROVNICA", "BRZOVICA", "DOBREPOLJE", "DOBROVA-POLHOV GRADEC", "DOL PRI LJUBLJANI", "DOMŽALE", "GROSUPLJE", "HORJUL", "IG", "IVANČNA GORICA", "KAMNIK", "KOMENDA", "LJUBLJANA", "LOG-DRAGOMER", "LOGATEC", "LUKOVICA", "MEDVODE", "MENGEŠ", "MORAVČE", "ŠKOFLJICA", "ŠMARTNO PRI LITIJI", "TRZIN", "VELIKE LAŠČE", "VODICE", "VRHNIKA","BREZOVICA")),
    "zasavska" : set(("HRASTNIK", "LITIJA", "TRBOVLJE", "ZAGORJE OB SAVI")),
    "jugovzhodnaslovenija" : set(("ČRNOMELJ", "DOLENJSKE TOPLICE", "KOČEVJE", "KOSTEL", "LOŠKI POTOK", "METLIKA", "MIRNA", "MIRNA PEČ", "MOKRONOG-TREBELNO", "NOVO MESTO", "OSILNICA", "RIBNICA", "SEMIČ", "SODRAŽICA", "STRAŽA", "ŠENTJERNEJ", "ŠENTRUPERT", "ŠKOCJAN", "ŠMARJEŠKE TOPLICE", "TREBNJE", "ŽUŽEMBERK")),
    "posavska" : set(("BISTRICA OB SOTLI", "BREŽICE", "KOSTANJEVICA NA KRKI", "KRŠKO", "RADEČE", "SEVNICA")),
    "savinjska" : set(("BRASLOVČE", "CELJE", "DOBJE", "DOBRNA", "GORNJI GRAD", "KOZJE", "LAŠKO", "LJUBNO", "LUČE", "MOZIRJE", "NAZARJE", "PODČETRTEK", "POLZELA", "PREBOLD", "REČICA OB SAVINJI", "ROGAŠKA SLATINA", "ROGATEC", "SLOVENSKE KONJICE", "SOLČAVA", "ŠENTJUR", "ŠMARJE PRI JELŠAH", "ŠMARTNO OB PAKI", "ŠOŠTANJ", "ŠTORE", "TABOR", "VELENJE", "VITANJE", "VOJNIK", "VRANSKO", "ZREČE", "ŽALEC")),
    "korska" : set(("ČRNA NA KOROŠKEM", "DRAVOGRAD", "MEŽICA", "MISLINJA", "MUTA", "PODVELKA", "PREVALJE", "RADLJE OB DRAVI", "RAVNE NA KOROŠKEM", "RIBNICA NA POHORJU", "SLOVENJ GRADEC", "VUZENICA")),
    "podravska" : set(("BENEDIKT", "CERKVENJAK", "CIRKULANE", "DESTRNIK", "DORNAVA", "DUPLEK", "GORIŠNICA", "HAJDINA", "HOČE-SLIVNICA", "JURŠINCI", "KIDRIČEVO", "KUNGOTA", "LENART", "LOVRENC NA POHORJU", "MAJŠPERK", "MAKOLE", "MARIBOR", "MARKOVCI", "MIKLAVŽ NA DRAVSKEM POLJU", "OPLOTNICA", "ORMOŽ", "PESNICA", "PODLEHNIK", "POLJČANE", "PTUJ", "RAČE-FRAM", "RUŠE", "SELNICA OB DRAVI", "SLOVENSKA BISTRICA", "SREDIŠČE OB DRAVI", "STARŠE", "SVETA ANA", "SV. TROJICA V SLOV. GORICAH", "SVETI ANDRAŽ V SLOV. GORICAH", "SVETI JURIJ V SLOV. GORICAH", "SVETI TOMAŽ", "ŠENTILJ", "TRNOVSKA VAS", "VIDEM", "ZAVRČ", "ŽETALE")),
    "pomurska" : set(("APAČE", "BELTINCI", "CANKOVA", "ČRENŠOVCI", "DOBROVNIK", "GORNJA RADGONA", "GORNJI PETROVCI", "GRAD", "HODOŠ", "KOBILJE", "KRIŽEVCI", "KUZMA", "LENDAVA", "LJUTOMER", "MORAVSKE TOPLICE", "MURSKA SOBOTA", "ODRANCI", "PUCONCI", "RADENCI", "RAZKRIŽJE", "ROGAŠOVCI", "SVETI JURIJ OB ŠČAVNICI", "ŠALOVCI", "TIŠINA", "TURNIŠČE", "VELIKA POLANA", "VERŽEJ"))
}
```

Branje podatkov Evidence trga nepremičnin. Podatki so zelo obsežni in so v večih datotekah. Razdeljeni so po letih od 2007 do 2022 in nato še za svako leto na datoteko šifrantov, datoteko poslov, datoteko zemljišč, datoteko delov stavb in datoteko strank, ki pa nam ni dostopna zaradi varovanja osebnih podatkov.


```python
def EtnRead(year, tip, header = None): #header je če nas zanima le header vrstica, določene ETN datoteke
    if tip not in ('posli','delistavb','zemljisca'): #table get check
        print(f'Unknown file name "{tip}";\nAllowed : "posli" , "delistavb", "zemljisca"')
        return None

    if not (2006 < year < 2023):  #year range check
        print(f'Incorrect year: {year}\nRange is [2007,2022]')
        return None
                            #index stolpca opombe, ki ga zbrišemo saj nam opomba lahko bloata np table
                            #index opombe v csvju : posli = zemljisca = 9 , delistavb = 27
                            ####če je treba preskočiti še kakšen stolpec, dodaj v pravi list index stolpca###
    indexi = [9]            
    if tip == 'delistavb':
        indexi = [27]
        
    if tip == 'zemljisca':
        pass        #preskoci


    data_path = f'podatki/ETN_SLO_CSV_A_KUP/ETN_SLO_KUP_{year}_20220326/ETN_SLO_KUP_{year}_{tip}_20220326.csv'

    with open(data_path,encoding='utf-8') as csvfile:
        reader = csv.reader(csvfile, delimiter=';')
        data = []

        head = next(reader,None) #skip header

        if header:
            return head #vrne header row

        for row in reader:
            try:
                for i in indexi:
                    row[i] = ""
            except Exception as e:
                print(row)
                print(e)
                return
            data.append(row)
    
    return np.array(data)

```


```python

#zemljisca2010 = EtnRead(2010,'zemljisca')
posli2010 = EtnRead(2010,'posli')
delistavb2010 = EtnRead(2010,'delistavb')

```


```python
#_ =[print(x) for x in enumerate(EtnRead(2010,'zemljisca',True))] 
#_ =[print(x) for x in enumerate(EtnRead(2010,'delistavb',True))]

```

Seznam stolpcev v datotekah ETN_SLO_KUP_{year}_delistavb_20220326.csv. Stoplci predstavljajo posamezne podatke o nepremičninah. Posamezne vrednosti so lahko tudi prazne, saj ni nujno, da ima vsaka nepremičnina atrij, kot primer.  


```python
def getDictPosli(posliData):
    dict_posli = {}
    for row in posliData:
        if row[4] == '' or row[4] == '0':  # cene 0 smiselne?
            continue
        dict_posli[row[0]] = float(row[4].replace(",","."))
    return dict_posli
```


```python
def getDictStavbe(delistavbData, tipiStavb = ('1','2') ):
    dict_stavbe = {}
    for row in delistavbData:
        if row[14] not in tipiStavb: #default: če stavba ni stanovalska hiša ali stanovanje jo preskoči
            continue

        povrsina = row[19] #stolpec 'Prodana površina'

        
        if povrsina == '': #ce ni podatka o povrsini
            continue

        povrsina = float(povrsina.replace(",",".")) 
        
        if povrsina == 0: #ce je povrsina 0
            continue

        obcina = row[3]
        
        dict_stavbe[row[0]] = (povrsina, obcina) 
        
        #(row[19],row[20],row[21],row[22])
    return dict_stavbe
```


```python
def generateCenaNaMeter(dictStavbe,dictPosli):
    kljuci = dictStavbe.keys()
    naMeter = []
    for kljuc in kljuci:
        if kljuc not in dictPosli:
            continue
        cena = dictPosli[kljuc]
        povrsina = dictStavbe[kljuc][0] # [0] nam da povrsino, [1] bi dal obcino
        naMeter.append(cena/povrsina)
    return (sum(naMeter) / len(naMeter),len(naMeter))
```


```python
def getAllCnM(): #get all cena na meter
    cenaNaMeter = {}
    for i in range(2007,2023):
        
        dictPosli = getDictPosli(EtnRead(i,'posli'))
        dictStavbe = getDictStavbe(EtnRead(i,'delistavb'))
        naMeter = generateCenaNaMeter(dictStavbe,dictPosli)
        cenaNaMeter[i] = naMeter[0]
        #print(f"Leto {i} cena na meter: {naMeter[0]} z stevilom meritev: {naMeter[1]}")
    return cenaNaMeter
    
```


```python
def whichRegija(stavba):
    for imeRegije,obcineRegije in regije.items():
        if stavba[1].upper() in obcineRegije:
            return imeRegije
    return "Brez"
```


```python
def generateCenaNaMeterPoRegiji(dictStavbe,dictPosli):
    stPoRegiji = { x : [] for x in regije.keys() }
    stPoRegiji["Brez"] = []

    kljuci = dictStavbe.keys()
    for kljuc in kljuci:
        if kljuc not in dictPosli:
            continue
        cena = dictPosli[kljuc]
        povrsina = dictStavbe[kljuc][0] # [0] nam da povrsino, [1] bi dal obcino
        regija = whichRegija(dictStavbe[kljuc])
        stPoRegiji[regija].append(cena/povrsina)

    for key in stPoRegiji:
        try:
            stPoRegiji[key] = ( sum(stPoRegiji[key]) / len(stPoRegiji[key]), len(stPoRegiji[key]))
        except ZeroDivisionError:
            stPoRegiji[key] = (-1, 0)

    return stPoRegiji
```


```python
def getRegijskiCenaNaMeter():
    cenaNaMeter = {}
    for i in range(2007,2023):
        dictPosli = getDictPosli(EtnRead(i,'posli'))
        dictStavbe = getDictStavbe(EtnRead(i,'delistavb'))
        naMeter = generateCenaNaMeterPoRegiji(dictStavbe,dictPosli)
        cenaNaMeter[i] = naMeter
        #print(f"Leto {i} cena na meter: {naMeter}")
    return cenaNaMeter
    
```


```python

regijskiMeter = getRegijskiCenaNaMeter()
```


```python
# struktura:
# dict let
#   dict regij
#       tuple(kvadratniMeter, stevilo meritev)
# 
# kvadratnimeter -1 pomeni brez podatka
print(regijskiMeter[2007]["osrednjeslovenska"][:]) #[0] = kvadratniMeter; [1] = stevilo meritev

```

    (3697.603513336031, 10)
    


```python
#Just use the tuple (2nd value) from CenaNaMeterPoRegiji

####### IGNORE FOR NOW ########

def getStPoRegiji(dictStavbe):
    stPoRegiji = { x : 0 for x in regije.keys() }
    stPoRegiji["Brez"] = 0

    for stavba in dictStavbe.values():
        stPoRegiji[whichRegija(stavba)] += 1
    
    return stPoRegiji
    
def getRegijaSteviloPoslov():
    letaRegiji = {}
    for i in range(2007,2023):
        dictStavbe = getDictStavbe(EtnRead(i,'delistavb'))
        stPoRegiji = getStPoRegiji(dictStavbe)
        letaRegiji[i] = stPoRegiji
    return letaRegiji

porazdelitevPoRegiji = getRegijaSteviloPoslov()
```


```python
cenaNaMeter = getAllCnM()
```

# Analiza podatkov


```python
allDataLenVsi = []
manka19Vsi = []
manka22Vsi = []
mankaObaVsi = []

for i in range(2007,2023):
    deliStavbData = EtnRead(i,'delistavb')

    #zakomentiraj spodnji dve vrstici če te zanimajo vsi podatki
    # in ne samo podatki o stanovanjih in stanovanskih hišah
    mask = np.logical_or(deliStavbData[:,14] == '1',deliStavbData[:,14] == '2')
    deliStavbData = deliStavbData[mask,:]

    allDataLen = deliStavbData.shape[0]
    manka19 = sum(deliStavbData[:,19] == '') / allDataLen
    manka22 = sum(deliStavbData[:,22] == '') / allDataLen
    mankaOba = sum(np.logical_and(deliStavbData[:,19] == '', deliStavbData[:,22] == '')) / allDataLen

    del deliStavbData

    allDataLenVsi.append(allDataLen)
    manka19Vsi.append(manka19)
    manka22Vsi.append(manka22)
    mankaObaVsi.append(mankaOba)

```


```python
fig = plt.figure()
fig.patch.set_facecolor('white')
plt.plot(range(2007,2023),allDataLenVsi)
plt.xlabel("Leto")
plt.ylabel("Število")
plt.title("Število podatkov o stavbah po letu")
plt.show()

fig = plt.figure()
fig.patch.set_facecolor('white')
plt.plot(range(2007,2023),manka19Vsi)
plt.xlabel("Leto")
plt.ylabel("Delež")
plt.title("Delež mankajočih podatkov o (19) 'Prodana površina'")
plt.show()

fig = plt.figure()
fig.patch.set_facecolor('white')
plt.plot(range(2007,2023),manka22Vsi)
plt.xlabel("Leto")
plt.ylabel("Delež")
plt.title("Delež mankajočih podatkov o (22) 'Prodana uporabna površina dela stavbe'")
plt.show()

fig = plt.figure()
fig.patch.set_facecolor('white')
plt.plot(range(2007,2023),mankaObaVsi)
plt.xlabel("Leto")
plt.ylabel("Delež")
plt.title("Delež mankajočih podatkov o (19) in (22)")
plt.show()
```


    
![png](projektna_files/projektna_37_0.png)
    



    
![png](projektna_files/projektna_37_1.png)
    



    
![png](projektna_files/projektna_37_2.png)
    



    
![png](projektna_files/projektna_37_3.png)
    



```python
from datetime import datetime

seznam = []
for leto in range(2014,2022):
    for mesec in range(1,13):
        seznam.append(datetime.strptime(str(leto)+"-"+str(mesec), '%Y-%m'))


matrika = np.array([[seznam,bruto,neto]], dtype = object)
minPlace =[]
for key in leto_min_placa.keys():
    if int(key) >=2014:
        minPlace.append(leto_min_placa[key])

minPlace = np.repeat(minPlace, 12)
minPlace = np. reshape(minPlace.astype(float), -1)
  
bruto1 =  np. reshape(matrika[:,1].astype(float), -1)
neto1 = np. reshape(matrika[:,2].astype(float), -1)
y = np. reshape(matrika[:,0].astype(datetime), -1)

fig = plt.figure()
fig.patch.set_facecolor('white')
plt.figure(figsize=(10,5))
plt.plot(y,bruto1, label="Bruto place")
plt.plot(y,neto1, label="Neto place")
plt.plot(y,minPlace,label ="Minimalne place")

plt.legend()
plt.xticks(rotation=90)
plt.show()

#print(y)
```


    <Figure size 432x288 with 0 Axes>



    
![png](projektna_files/projektna_38_1.png)
    


Graf prikazuje obnašanje prihodkov skozi obdobje 2014 do 2022. Razberemo lahko, da prihodki počasi rastejo, tako povprečna, kot tudi minimalna, kar je logično. Opažamo lahko skoke v koncih vsakega leta. Te skoke najverjetneje povzročijo božični bonusi.


```python
fig = plt.figure()
fig.patch.set_facecolor('white')
fig, ax = plt.subplots(figsize=(10,7))
ax.set_title("Spreminjanje stevila transakcij in njihovih skupnih vrednosti po četrtletjih")
ax.tick_params('x', labelrotation=90)
ax.set_xlabel('Cas')
ax.set_ylabel('Stevilo transakcij', color='red')
# Plot linear sequence, and set tick labels to the same color
ax.plot(xVrednosti,stareTransakcije, color='red')
ax.plot(noveTransakcije, color='red')
ax.tick_params(axis='y', labelcolor='red')

# Generate a new Axes instance, on the twin-X axes (same position)
ax2 = ax.twinx()

# Plot exponential sequence, set scale to logarithmic and change tick color
ax2.set_ylabel('Vrednost transakcij', color='green')
ax2.plot(stareVrednost, color='green')
ax2.plot(noveVrednost, color='green')
ax2.tick_params(axis='y', labelcolor='green')


plt.show()
```


    <Figure size 432x288 with 0 Axes>



    
![png](projektna_files/projektna_40_1.png)
    


Zgornji graf prikazuje rast števila transakcij in njihovih skupnih vrednosti po četrtletjih od leta 2010 do leta 2021. Spnodji del grafa prikazuje podatke za rabljene nepremičnine, zgonrji del pa podatke za nove nepremičnine. Zeleni črti prikazujeta vrednosti transakcij, rdeči lrti pa število transakcij. Razberemo lahko, da je število transakcij novih nepremičnin veliko večje, kot starih. Lahko sklepamo, da se gradnja novih nepremičnin veča, ali pa da se stare nepremičnine ne prodajajo. Lahko sta pa tudi oba primera razlog za razmik. Razberemo lahko tudo da vrednosti sledita trendom števila transakcij. Če število transakcij raste, raste tudi skupna vrednost, kar je logično. Razberljiva sta tudi padca, še predvsem pri novih nepremičninah v času dveh večjih valov okuženosti z COVID-om. Takrat je prodaja padla še posebej pa se padec pozna pri novih nepremičninah, kjer je najverjetneje padla tudi gradnja novih nepremičnin. Kljub padcem se graf novih premičnin hitro vrne nazaj, k svojem trendu naraščanja. Glede rabljenih nepremičnin pa lahko vidimo, da sta od 2010 do 2014 število ter vrednost padala, nato naraščala do leta 2018, in sedaj spet padata. Iz zgornjega dela, ki prikazuje podatke za nove nepremičnine lahko opazimo, da je skupna vrednost transkacij bolj narasla kot število transakcij. Iz tega lahko sklepamo, da se je cena posameznih transakcij povečala.

Dodatne funkcije


```python
def sizeof_fmt(num, suffix="B"): #funkcija za formatiranje izpisa velikosti spremenljivk
    for unit in ["", "Ki", "Mi", "Gi", "Ti", "Pi", "Ei", "Zi"]:
        if abs(num) < 1024.0:
            return f"{num:3.1f}{unit}{suffix}"
        num /= 1024.0
    return f"{num:.1f}Yi{suffix}"

#preveri katerih podatkov ne more prebrati 
def checkRAM(ime):
    for i in range(2007,2023):
        try:
            data = EtnRead(i,ime)
            size = sizeof_fmt(sys.getsizeof(data))
            print(f"{i} => {data.dtype}, ram usage: {size}") #preveri tip tabele ie string length
            del data
        except Exception as e:
            print(f"{i} => {e}")  #prikazi katero leto ti vrže (memoryError -> zmankal rama) exception 
            pass

def test():
    print('delistavb')
    checkRAM('delistavb')
    print()

    print('zemljisca')
    checkRAM('zemljisca')
    print()

    print('posli')
    checkRAM('posli')
    
#test()
```


```python
def _zemljisca():       #fix ce mislis use
    dict_zemljisca = {}
    for row in zemljisca2010:
        if row[10] == '' or int(row[10]) == '0': # če ni površine preskoči
            continue
        if row[5] not in (1,2,3,4): #ce ni primernega tipa za stanovanje oz hišo nas ne zanima
            pass
            #continue
        if "/" in row[8]: # row[8] != 'NP' and row[8] != '6179844811' and row[8] != '': #preveri pomen NP, ali je Ni Podatka ali kaj
            ops = row[8].split("/") 
        else:
            ops = ['1', '1']
        
        if ops[0] == '':
            ops[0] = 1
        if ops[1] == '':
            ops[1] = 1

        prodanDelez = int(ops[0]) / int(ops[1])

        if prodanDelez == 0:
            continue


        povrsinaParcele = float(row[10].replace(",","."))
        if row[0] in dict_zemljisca:
            dict_zemljisca[row[0]]+= povrsinaParcele #* prodanDelez
        else:
            dict_zemljisca[row[0]] = povrsinaParcele #* prodanDelez
```

# Zaključek

Do sedaj smo pregledali podatje o plačah in nepremičninah. Vidimo rast v obeh primerih, vendar če hočemo odgovoriti na naše vprašanje moramo to rast še dodatno primerjati med seboj. V nadaljevanju projektne naloge želimo bolje pregledati trende pri nepremičninah, kar bomo storili s pomočjo podatkov evidence trga nepremičnin. Želeli bi boljšo predstavitev glede na regije in boljšo razdelitev glede na tip nepremičnine.

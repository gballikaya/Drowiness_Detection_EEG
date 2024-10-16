# Drowiness_Detection_EEG

[Kaynak](https://www.mathworks.com/help/wavelet/ug/time-frequency-convolutional-network-for-eeg-data-classification.html)

### step 1 . [Download dataset:](https://www.upf.edu/web/ntsa/downloads/-/asset_publisher/xvT6E4pczrBw/content/2001-indications-of-nonlinear-deterministic-and-finite-dimensional-structures-in-time-series-of-brain-electrical-activity-dependence-on-recording-regi?inheritRedirect=false&redirect=https://www.upf.edu/web/ntsa/downloads?p_p_id%3D101_INSTANCE_xvT6E4pczrBw%26p_p_lifecycle%3D0%26p_p_state%3Dnormal%26p_p_mode%3Dview%26p_p_col_id%3Dcolumn-1%26p_p_col_count%3D1#.X5Ep-S337UI)

Z.zip (A), Gözleri açık normal denekler

O.zip (B), Gözleri kapalı normal denekler

N.zip (C), Epilepsili hastalardan nöbetsiz kayıtlar. Epileptojenik bölgenin karşısındaki yarım küredeki hipokampüsten elde edilen kayıtlar

F.zip (D), Epilepsili hastalardan nöbetsiz kayıtlar. Epileptojenik bölgeden elde edilen kayıtlar.  

S.zip (E), Epilepsili hastalardan nöbet aktivitesi gösteren kayıtlar.



Örnekleme frekansı: 173.61 Hz

Kayıt süresi: 23.6 sn

Her zaman serisi 4097 örnek içermektedir.

MATLAB:
zip dosyaları içerisindeki txt dosyaları çıkartılıyor. toplam 500 adet txt dosyası. her bir txt dosyası içerisinde 4097 sayısal değer var
'''Her TXT dosyası, ASCII kodunda bir EEG zaman serisinin 4096 örneğinden oluşur.'''
```
parentDir = 'C:\Users\bgamz\Desktop\bonn_classification';
cd(parentDir)
dataDir = parentDir;
unzip("z.zip",dataDir)
unzip("o.zip",dataDir)
unzip("n.zip",dataDir)
unzip("f.zip",dataDir)
unzip("s.zip",dataDir)
```
dataDir altında bulunan tüm .txt dosyalarını içeren bir tabular text datastore oluşturur. Ardından, __MACOSX ifadesini içeren dosyaları filtreleyerek bu dosyaları datastore'dan çıkarır. 

```
tds = tabularTextDatastore(dataDir, "IncludeSubfolders", true, "FileExtensions", ".txt"); (1*1 TabularTextDatastore)
extraTXT = contains(tds.Files, "__MACOSX"); (500*1 logical) 
tds.Files(extraTXT) = [];
labels = filenames2labels(tds.Files,"ExtractBetween",[1 1]);  (500*1 categorical)
```
tabularTextDatastore içindeki tüm verileri okur ve her bir EEG zaman serisini bir hücre dizisine (eegData) kaydeder. Ayrıca, her okunan veriden sonra bir sayaç artırılır. En son olarak, datastore'un konumu sıfırlanır, böylece veriler tekrar okunabilir. 

eeg verileri : satır vektörlerinin bir hücre dizisi haline getirilmiş
```
ii = 1;
eegData = cell(numel(labels),1); (500*1 cell)
while hasdata(tds)
    tsTable = read(tds); (4097*1 table)
    ts = tsTable.Var1; (4097*1 double)
    eegData{ii} = reshape(ts,1,[]);  
    ii = ii+1;
end
```
Z ve O etiketlerini (gözleri açık ve kapalı epilepsi olmayan kişiler) "Normal" olarak gruplandırmaktır. 

F ve N "Nöbet Öncesi" olarak gruplandırmaktır. 

S (nöbet aktivitesi olan epilepsili kişilerde elde edilen kayıtlar) "Nöbet" olarak adlandırılmaktadır.

```
labels3Class = labels;   (labels de olduğu gibi 500*1 categorical)
labels3Class = removecats(labels3Class,["F","N","O","S","Z"]);
labels3Class(labels == categorical("Z") | labels == categorical("O")) = ...
    categorical("Normal");
labels3Class(labels == categorical("F") | labels == categorical("N")) = ...
    categorical("Pre-seizure");
labels3Class(labels == categorical("S")) = categorical("Seizure");
```
veri dağılımı
```
     Normal           200 
     Pre-seizure      200 
     Seizure          100
```
%70 eğitim, %20 test ve %10 olarak bölündü

Bu aşamada verilerin son haline bakalım

etiketler labels3Class değişkeninde 500*1 categorical string verilerle kaydedilmiş

eeg verileri ise eegData içerisinde 500*1 cell ve her bir hücrede 1*4097 double veriler var 

```
idxSPN = splitlabels(labels3Class,[0.7 0.2 0.1]);
trainDataSPN = eegData(idxSPN{1});
trainLabelsSPN = labels3Class(idxSPN{1});
testDataSPN = eegData(idxSPN{2});
testLabelsSPN = labels3Class(idxSPN{2});
validationDataSPN = eegData(idxSPN{3});
validationLabelsSPN = labels3Class(idxSPN{3});
```
Bu aşamadan sonra eğitime geçiliyor.

Öncelikle elimizdeki verileri bu formata uygun hale getirelim

[mit-bih-polysomnographic-database kullanılacaktır.] (https://physionet.org/content/slpdb/1.0.0/)

Veritabanı, EKG sinyallerini ve EEG ile solunum sinyallerinin uyku evreleri ve apne açısından açıklandığı 80 saatten fazla dört, altı ve yedi kanallı polisomnografik kayıt içerir. 

Veritabanı, her biri 4 dosya içeren 18 kayıttan oluşmaktadır: slp01a, slp01b, slp02a, slp02b, slp03, slp04, slp14, slp16, slp32,slp37, slp41, slp45, slp48, slp59, slp60, slp61, slp66, slp67. dosya türleri: .hea, .ecg, .dat, .st, .xws

Uyku/apne açıklamaları: .st dosyaları
Beat açıklamaları: .ecg dosyaları
Sinyaller: .dat dosyaları
Başlık: .hea dosyaları

Her kayıt , sinyal tipleri, kalibrasyon sabitleri, kayıt uzunluğu ve (dosyanın son satırında) deneklerin yaşı, cinsiyeti ve kilosu (kg cinsinden) hakkında bilgi içeren kısa bir metin dosyası olan bir başlık (.hea) dosyası içerir. Bu veritabanında, 16 denek de erkekti, yaşları 32 ila 56 arasındaydı (ortalama yaş 43) ve kiloları 89 ila 152 kg arasında değişiyordu (ortalama kilo 119 kg). Slp01a ve slp01b kayıtları , yaklaşık bir saatlik bir boşlukla ayrılmış bir deneğin polisomnogramının bölümleridir; slp02a ve slp02b kayıtları , on dakikalık bir boşlukla ayrılmış başka bir deneğin polisomnogramının bölümleridir. Kalan 14 kayıt da farklı deneklere aittir.

Tüm kayıtlar bir EKG sinyali, invaziv kan basıncı sinyali, bir EEG sinyali ve bir solunum sinyali içerir. Altı ve yedi kanallı kayıtlar ayrıca indüktans pletismografisi ile türetilen bir solunum çabası sinyali içerir; bazıları bir EOG sinyali ve bir EMG sinyali içerir ve geri kalanlar bir kardiyak atım hacmi sinyali ve bir kulak memesi oksimetre sinyali içerir. 

18 kaydın tamamı EEG verisi içeriyor.
```
aux	meaning
W	subject is awake
1	sleep stage 1
2	sleep stage 2
3	sleep stage 3
4	sleep stage 4
R	REM sleep
H	Hypopnea
HA	Hypopnea with arousal
OA	Obstructive apnea
X	Obstructive apnea with arousal
CA	Central apnea
CAA	Central apnea with arousal
L	Leg movements
LA	Leg movements with arousal
A	Unspecified arousal
MT	Movement time
```

veriyi incelemek için [ilgili linkteki](https://physionet.org/content/wfdb-matlab/0.10.0/) adımları takip ederek MATLAB WFDB (Waveform Database) paketini kuralım.

WFDB paketinin [kaynak kodu](https://github.com/ikarosilva/wfdb-app-toolbox) GİTHUB'da açık erişimlidir. Detaylara ulaşılabilir.
```
örneğin slp66 kaydı 3:40 (3 saat 40dk) ve veri boyutu 3300000*7 ve 439 anotasyon : bunu doğrulayalım :

sinyalden saniyede 250 örnek alınıyor

30 saniyelik epochlar ile analiz ediliyor

her 30 sn'de bir etiket atanıyor

30 saniyede alınan örnek sayısı: 30*250=7500

7500*439=3300000 örnek

3 saat 40 dk=220 dakika=13200 saniye

13200*250=3300000 örnek
```
```
bir başka örnek slp48 kaydı 6:20 ve veri boyutu 5700000*7 ve 760 anotasyona sahip: bunu doğrulayalım:

sinyalden saniyede 250 örnek alınıyor

30 saniyelik epochlar ile analiz ediliyor

her 30 sn'de bir etiket atanıyor

30 saniyede alınan örnek sayısı: 30*250=7500

760*7500=5700000 örnek

6 saat 20 dakika=22800 saniye

22800*250=5700000 örnek 

```






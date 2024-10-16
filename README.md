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


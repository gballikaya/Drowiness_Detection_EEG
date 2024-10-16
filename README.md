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

klasör yapısı

bonn_classification  (C:\Users\bgamz\Desktop\bonn_classification)

  boon_cls.m
  
  F.zip
  
  S.zip
  
  O.zip
  
  N.zip
  
  Z.zip

MATLAB:
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



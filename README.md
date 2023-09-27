# webscraping-#

#The outputs were shared with Excel.#
#The process is to properly create a dataframe by pulling the prices and features of all houses in Istanbul on the Emlakjet website.#
from selenium import webdriver
from selenium.webdriver.common.by import By
import pandas as pd

# Chrome web sürücüsünü başlat
browser = webdriver.Chrome()

# Belirtilen URL'ye git
browser.get("https://www.emlakjet.com/satilik-konut/istanbul/")

# Sayfa yüklenene kadar bekleyebilirsiniz
# Örnek olarak, 5 saniye bekleyelim
browser.implicitly_wait(5)

# Detaylar, fiyatlar ve konumlar için boş liste oluşturun
detaylar = []
fiyatlar = []
konumlar = []

# İlk 50 sayfa için döngüyü ayarlayın
i = 1
while i <= 50:
    # Belirtilen CSS seçiciler ile eşleşen öğeleri bulun
    hk = browser.find_elements(By.CSS_SELECTOR, '._2UELHn')
    halil = browser.find_elements(By.CSS_SELECTOR, ' ._2C5UCT')
    konum = browser.find_elements(By.CSS_SELECTOR, '._2wVG12')

    # Detaylar, fiyatlar ve konumları listelere ekleyin
    for detay, fiyat, konum in zip(hk, halil, konum):
        detaylar.append(detay.text)
        fiyatlar.append(fiyat.text)
        konumlar.append(konum.text)

    
    i += 1

# Web sürücüyü kapat
browser.quit()




# Verileri bir Pandas DataFrame'e dönüştürün
df = pd.DataFrame({'Detaylar': detaylar, 'Fiyatlar': fiyatlar, 'Konumlar': konumlar})
df['Metrekare'] = df['Detaylar'].str.extract(r'(\d+\s+m2)')

df['Kat Sayısı'] = df['Detaylar'].str.extract(r'(\d+)\. Kat')
# Oda sayısını çıkar (Örneğin, "4+1" gibi)
df['Oda Sayısı'] = df['Detaylar'].str.extract(r'(\d+\+\d+)')

# İlan tarihini çıkar (Örneğin, "11 Eylül" gibi)
df['İlan Tarihi'] = df['Detaylar'].str.extract(r'(\d+\s+Eylül)')
df['Il'] = df['Konumlar'].str.split('-').str[0]
df['Ilce'] = df['Konumlar'].str.split('-').str[1]
df['Semt'] = df['Konumlar'].str.split('-').str[2]
df.drop('Detaylar', axis=1, inplace=True)
df.drop('Konumlar', axis=1, inplace=True)

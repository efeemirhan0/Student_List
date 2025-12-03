#include <iostream>
#include <vector>
#include <string>
#include <numeric>
#include <limits>
#include <iomanip>
#include <fstream>

// Verilerin kaydedileceği dosyanın adı
const std::string DOSYA_ADI = "ogrenci-listesi.txt";

//Her öğrenciye ait struct
struct Ogrenci
{
    std::string ad;
    std::vector<double> notlar; //Her öğrencinin kendine ait not vektörü
};
 
double ortHesapla(const std::vector<double>& notlar) {
    if (notlar.empty()) {
    return 0.0;
    }
double toplam = std::accumulate(notlar.begin(), notlar.end (),0.0);
return toplam / notlar.size();
}
//Programın çökmemesine yarıyor
void temizleGiris() {
    std::cin.clear(); // Hata bayrağını temizler
    // Giriş tamponunda kalan geçersiz veriyi entera kadar temizler
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
}
int ogrenciBul(const std::vector<Ogrenci>& ogrenciler, const std::string& ad) {
    for (int i = 0; i < ogrenciler.size(); ++i) {
        if (ogrenciler[i].ad == ad) {
            return i; // Öğrenci buldu, indeksini döndürür
        }
    }
    return -1; // Öğrenci bulunmuyor
}
void ogrenciEkle(std::vector<Ogrenci>& ogrenciler) {
    Ogrenci yeniOgrenci;
    std::cout << "Eklenecek ogrencinin adini girin: ";
    std::cin.ignore(); 
    std::getline(std::cin, yeniOgrenci.ad);

    // Aynı isimde öğrenci var mı diye kontrol eder
    if (ogrenciBul(ogrenciler, yeniOgrenci.ad) != -1) {
        std::cout << "Bu isimde bir ogrenci zaten kayitli." << std::endl;
        return;
    }

    ogrenciler.push_back(yeniOgrenci); // Yeni öğrenciyi ana vektöre ekler
    std::cout << "'" << yeniOgrenci.ad << "' eklendi." << std::endl;
}
void ogrenciCikar(std::vector<Ogrenci>& ogrenciler) {
    std::string ad;
    std::cout << "Çikarilacak ogrencinin adi: ";
    std::cin.ignore(); 
    std::getline(std::cin, ad);

    int index = ogrenciBul(ogrenciler, ad);

    if (index != -1) {
        // Öğrenciyi vektörden siler
        ogrenciler.erase(ogrenciler.begin() + index);
        std::cout << "'" << ad << "' silindi." << std::endl;
    } else {
        std::cout << "Ogrenci yok" << std::endl;
    }
}
void puanGirisi(std::vector<Ogrenci>& ogrenciler) {
    std::string ad;
    std::cout << "Not girilecek ogrencinin adi: ";
    std::cin.ignore();
    std::getline(std::cin, ad);

    int index = ogrenciBul(ogrenciler, ad);

    if (index != -1) {
        double notu;
        std::cout << "Girilecek not (0-100): ";
        
        while (!(std::cin >> notu) || notu < 0 || notu > 100) {
            std::cout << "0-100 arasi bir sayi girin: ";
            temizleGiris(); 
        }

        ogrenciler[index].notlar.push_back(notu);
        std::cout << "Not eklendi." << std::endl;

        double ort = ortHesapla(ogrenciler[index].notlar);
        
        std::cout << "'" << ogrenciler[index].ad 
                  << "' adli ogrencinin yeni not ortalamasi: " 
                  << std::fixed << std::setprecision(2) << ort << std::endl;
    } else {
        std::cout << "Ogrenci bulunamadi" << std::endl;
    }
}
void tumOgrencileriListele(const std::vector<Ogrenci>& ogrenciler) {
    if (ogrenciler.empty()) {
        std::cout << "Sistemde kayiitli ogrenci yok." << std::endl;
        return;
    }

    std::cout << "\nOgrenci Listesi" << std::endl;

    std::cout << std::fixed << std::setprecision(2);

    for (const Ogrenci& ogr : ogrenciler) {
        std::cout << "Ogrenci: " << ogr.ad << std::endl;

        std::cout << "  Notlari: ";
        if (ogr.notlar.empty()) {
            std::cout << "Not girilmemis.";
        } else {
            for (double notu : ogr.notlar) {
                std::cout << notu << " | ";
            }
        }
        std::cout << std::endl; 

        double ort = ortHesapla(ogr.notlar);
        std::cout << "  Ortalamasi: " << ort << std::endl;
        std::cout << "                       " << std::endl;
    }
}
void ogrenciAra(const std::vector<Ogrenci>& ogrenciler) {
    std::string ad;
    std::cout << "Aranacak ogrencinin adi: ";
    std::cin.ignore();
    std::getline(std::cin, ad);

    int index = ogrenciBul(ogrenciler, ad);

    if (index != -1) {
        std::cout << "Ogrenci bulundu" << std::endl;
        const Ogrenci& ogr = ogrenciler[index]; 

        std::cout << std::fixed << std::setprecision(2); 
        std::cout << "Ogrenci: " << ogr.ad << std::endl;
        std::cout << "  Notlari: ";
        if (ogr.notlar.empty()) {
            std::cout << "Not girilmemis.";
        } else {
            for (double notu : ogr.notlar) {
                std::cout << notu << " | ";
            }
        }
        std::cout << std::endl;
        double ort = ortHesapla(ogr.notlar);
        std::cout << "  Ortalamasi: " << ort << std::endl;
    } else {
        std::cout << "Ogrenci bulunamadi" << std::endl;
    }
}

void verileriKaydet(const std::vector<Ogrenci>& ogrenciler) {
    std::ofstream dosya(DOSYA_ADI);

    if (!dosya.is_open()) {
        std::cerr << "Hata: Kayit dosyasi açilamadi" << std::endl;
        return;
    }

    for (const Ogrenci& ogr : ogrenciler) {
        dosya << ogr.ad << std::endl;
        dosya << ogr.notlar.size() << std::endl;
        for (double notu : ogr.notlar) {
            dosya << notu << std::endl;
        }
    }

    dosya.close();
    std::cout << "[Sistem: Veriler '" << DOSYA_ADI << "' dosyasina kaydedildi.]" << std::endl;
}

void verileriYukle(std::vector<Ogrenci>& ogrenciler) {
    std::ifstream dosya(DOSYA_ADI);

    if (!dosya.is_open()) {
        std::cout << "[Sistem: Kayit dosyasi bulunamadi. Yeni kayit oluşturuluyor.]" << std::endl;
        return;
    }

    ogrenciler.clear(); 
    std::string ad;
    int notSayisi;

    while (std::getline(dosya, ad)) {
        Ogrenci tempOgrenci;
        tempOgrenci.ad = ad;

        if (!(dosya >> notSayisi)) {
            std::cerr << " Dosya formati bozuk." << std::endl;
            break; 
        }

        for (int i = 0; i < notSayisi; ++i) {
            double notu;
            if (!(dosya >> notu)) {
                std::cerr << " Dosya formati bozuk. " << std::endl;
                break;
            }
            tempOgrenci.notlar.push_back(notu);
        }

        dosya.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
        ogrenciler.push_back(tempOgrenci);
    }

    dosya.close();
    std::cout << "[Sistem: " << ogrenciler.size() << " öğrenci kaydi yüklendi.]" << std::endl;
}



int main() {
    
    std::vector<Ogrenci> tumOgrenciler;

    verileriYukle(tumOgrenciler);

    int secim = -1; 

    do {
        std::cout << "\nOgrenci Not Sistemi" << std::endl;
        std::cout << "1. Ogrenci Ekle" << std::endl;
        std::cout << "2. Ogrenci Cikar" << std::endl;
        std::cout << "3. Ogrenciye Puan Girisi" << std::endl;
        std::cout << "4. Tum Ogrencileri Listele" << std::endl;
        std::cout << "5. Ogrenci Ara" << std::endl;
        std::cout << "0. Cikis" << std::endl;
        std::cout << "                       " << std::endl;
        std::cout << "Seciminiz: ";

        
        while (!(std::cin >> secim)) {
            std::cout << "Gecersiz sayi. Menuden bir sayi girin: ";
            temizleGiris();
        }

        
        switch (secim) {
            case 1:
                ogrenciEkle(tumOgrenciler);
                verileriKaydet(tumOgrenciler);
                break;
            case 2:
                ogrenciCikar(tumOgrenciler);
                verileriKaydet(tumOgrenciler); 
                break;
            case 3:
                puanGirisi(tumOgrenciler);
                verileriKaydet(tumOgrenciler); 
                break;
            case 4:
                tumOgrencileriListele(tumOgrenciler);
                break;
            case 5:
                ogrenciAra(tumOgrenciler);
                break;
            case 0:
                std::cout << "Programdan cikiliyor..." << std::endl;
                break;
            default:
                std::cout << "0-5 arasi bir sayi girin." << std::endl;
        }

    } while (secim != 0);

    return 0; 
}

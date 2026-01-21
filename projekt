import sys
from PyQt5.QtWidgets import QApplication, QWidget, QPushButton, QLabel, QGridLayout
from PyQt5.QtCore import Qt, QTimer, QPointF
from PyQt5.QtGui import QPainter, QColor, QPen, QPainterPath

print("Uruchomiono symulację systemu.")

#Klasy glowne symulacji

class Rura:
    def __init__(self, punkty, grubosc=12, kolor=Qt.gray, kolor_cieczy=QColor(0, 180, 255)):
        self.punkty = [QPointF(float(p[0]), float(p[1])) for p in punkty]
        self.grubosc = grubosc
        self.kolor_rury = kolor
        self.kolor_cieczy = kolor_cieczy
        self.czy_plynie = False

    def ustaw_przeplyw(self, plynie):
        self.czy_plynie = plynie

    def draw(self, painter):
        if len(self.punkty) < 2: return
        path = QPainterPath()
        path.moveTo(self.punkty[0])
        for p in self.punkty[1:]: path.lineTo(p)
        
        # 1. Obudowa
        painter.setPen(QPen(self.kolor_rury, self.grubosc, Qt.SolidLine, Qt.RoundCap, Qt.RoundJoin))
        painter.setBrush(Qt.NoBrush)
        painter.drawPath(path)

        # 2. Ciecz 
        if self.czy_plynie:
            painter.setPen(QPen(self.kolor_cieczy, self.grubosc-4, Qt.SolidLine, Qt.RoundCap, Qt.RoundJoin))
            painter.drawPath(path)

class Zbiornik:
    def __init__(self, x, y, width=100, height=140, nazwa=""):
        self.x = x; self.y = y
        self.width = width; self.height = height
        self.nazwa = nazwa
        self.pojemnosc = 100.0
        self.aktualna_ilosc = 0.0
        self.poziom = 0.0

    def dodaj_ciecz(self, ilosc):
        wolne = self.pojemnosc - self.aktualna_ilosc
        dodano = min(ilosc, wolne)
        self.aktualna_ilosc += dodano
        self.aktualizuj_poziom()
        return dodano

    def usun_ciecz(self, ilosc):
        usunieto = min(ilosc, self.aktualna_ilosc)
        self.aktualna_ilosc -= usunieto
        self.aktualizuj_poziom()
        return usunieto

    def aktualizuj_poziom(self):
        self.poziom = self.aktualna_ilosc / self.pojemnosc

    def czy_pusty(self): return self.aktualna_ilosc <= 0.1
    def czy_pelny(self): return self.aktualna_ilosc >= self.pojemnosc - 0.1
    def punkt_gora_srodek(self): return (self.x + self.width/2, self.y)
    def punkt_dol_srodek(self): return (self.x + self.width/2, self.y + self.height)
    
    def draw(self, painter):
        # Ciecz
        if self.poziom > 0:
            h = self.height * self.poziom
            painter.setPen(Qt.NoPen)
            painter.setBrush(QColor(0, 120, 255, 200))
            painter.drawRect(int(self.x+3), int(self.y+self.height-h), int(self.width-6), int(h-2))
        # Obrys
        painter.setPen(QPen(Qt.white, 4, Qt.SolidLine, Qt.RoundCap, Qt.MiterJoin))
        painter.setBrush(Qt.NoBrush)
        painter.drawRect(int(self.x), int(self.y), int(self.width), int(self.height))
        # Podpis
        painter.setPen(Qt.white)
        painter.drawText(int(self.x), int(self.y-10), self.nazwa)

class Zbiornik_para(Zbiornik): 
    def punkt_gora_srodek_P(self): return (self.x + self.width/2, self.y)
    def punkt_dol_srodek_P(self): return (self.x + self.width/2, self.y + self.height)

class pompa:
    def __init__(self, x, y, width=24, height=14, nazwa=""):
        self.x = x; self.y = y; self.width = width; self.height = height; self.nazwa = nazwa
    def draw(self, painter):
        kolor_pompy = QColor(0, 255, 0)
        painter.setPen(QPen(kolor_pompy, 2))
        painter.setBrush(Qt.NoBrush)
        painter.drawRect(int(self.x), int(self.y), int(self.width), int(self.height))
        painter.setPen(kolor_pompy)
        painter.drawText(int(self.x), int(self.y - 5), self.nazwa)

class grzanie:
    def __init__(self, x, y, width=40, height=20, nazwa=""):
        self.x = x; self.y = y; self.width = width; self.height = height; self.nazwa = nazwa
        self.czy_grzeje = False
        
    def draw(self, painter):
        kolor = QColor(255, 100, 0) if self.czy_grzeje else QColor(100, 100, 100)
        grubosc = 3 if self.czy_grzeje else 2
        painter.setPen(QPen(kolor, grubosc, Qt.SolidLine, Qt.RoundCap, Qt.RoundJoin))
        painter.setBrush(Qt.NoBrush)
        
        path = QPainterPath()
        start_x = self.x; start_y = self.y + self.height / 2
        path.moveTo(start_x, start_y)
        step = self.width / 4
        path.lineTo(start_x + step * 0.5, self.y)
        path.lineTo(start_x + step * 1.5, self.y + self.height)
        path.lineTo(start_x + step * 2.5, self.y)
        path.lineTo(start_x + step * 3.5, self.y + self.height)
        path.lineTo(start_x + self.width, start_y)
        
        painter.drawPath(path)
        painter.setPen(Qt.white)
        painter.drawText(int(self.x), int(self.y + self.height + 15), self.nazwa)

class para:
    def __init__(self, x, y, width, height):
        self.x = x; self.y = y; self.width = width; self.height = height; self.widoczna = False
    def draw(self, painter):
        pass 

#Panel sterowania

class PanelSterowania(QWidget):
    def __init__(self, symulacja):
        super().__init__()
        self.symulacja = symulacja
        self.setWindowTitle("Panel Sterowania")
        self.setFixedSize(400, 600)
        self.setStyleSheet("background-color: #333;")
        
        layout = QGridLayout()
        layout.setSpacing(10)
        
        controls = [
            ("Zbiornik 1", self.symulacja.dodaj, self.symulacja.odejmij, 0),
            ("Zbiornik 2", self.symulacja.dodaj2, self.symulacja.odejmij2, 1),
            ("Zbiornik 3", self.symulacja.dodaj3, self.symulacja.odejmij3, 2),
            ("Zbiornik Para 1", self.symulacja.dodaj_zp1, self.symulacja.odejmij_zp1, 3),
            ("Zbiornik Para 2", self.symulacja.dodaj_zp2, self.symulacja.odejmij_zp2, 4),
        ]

        for nazwa, func_plus, func_minus, row in controls:
            btn_p = QPushButton(f"{nazwa} +", self)
            btn_p.setStyleSheet("background-color: #444; color: white; padding: 5px;")
            btn_p.clicked.connect(func_plus)
            layout.addWidget(btn_p, row, 0)
            
            btn_m = QPushButton(f"{nazwa} -", self)
            btn_m.setStyleSheet("background-color: #444; color: white; padding: 5px;")
            btn_m.clicked.connect(func_minus)
            layout.addWidget(btn_m, row, 1)

        self.btn_start = QPushButton("Start / Stop Symulacji", self)
        self.btn_start.setStyleSheet("background-color: #555; color: white; font-weight: bold; padding: 10px;")
        self.btn_start.clicked.connect(self.symulacja.przelacz_symulacje)
        layout.addWidget(self.btn_start, 5, 0, 1, 2)

        self.btn_pompy = QPushButton("Pompy ON/OFF", self)
        self.btn_pompy.setStyleSheet("background-color: red; color: white; font-weight: bold; padding: 10px;")
        self.btn_pompy.clicked.connect(self.obsluga_pomp)
        layout.addWidget(self.btn_pompy, 6, 0, 1, 2)
        
        self.btn_grzanie = QPushButton("Grzanie ON/OFF", self)
        self.btn_grzanie.setStyleSheet("background-color: red; color: white; font-weight: bold; padding: 10px;")
        self.btn_grzanie.clicked.connect(self.obsluga_grzania)
        layout.addWidget(self.btn_grzanie, 7, 0, 1, 2)

        self.setLayout(layout)

    def obsluga_pomp(self):
        self.symulacja.praca_pomp()
        color = "green" if self.symulacja.on else "red"
        self.btn_pompy.setStyleSheet(f"background-color: {color}; color: white; font-weight: bold; padding: 10px;")

    def obsluga_grzania(self):
        self.symulacja.podgrzewanie(self.btn_grzanie)


# --- GŁÓWNA SYMULACJA ---

class SymulacjaKaskady(QWidget):
    
    # Metody sterujące
    def dodaj(self): self.z1.dodaj_ciecz(100 - self.z1.aktualna_ilosc); self.update()
    def odejmij(self): self.z1.dodaj_ciecz(-self.z1.aktualna_ilosc); self.update()
    def dodaj2(self): self.z2.dodaj_ciecz(100 - self.z2.aktualna_ilosc); self.update()
    def odejmij2(self): self.z2.dodaj_ciecz(-self.z2.aktualna_ilosc); self.update()
    def dodaj3(self): self.z3.dodaj_ciecz(100 - self.z3.aktualna_ilosc); self.update()
    def odejmij3(self): self.z3.dodaj_ciecz(-self.z3.aktualna_ilosc); self.update()
    def dodaj_zp1(self): self.zp1.dodaj_ciecz(100 - self.zp1.aktualna_ilosc); self.update()
    def odejmij_zp1(self): self.zp1.dodaj_ciecz(-self.zp1.aktualna_ilosc); self.update()
    def dodaj_zp2(self): self.zp2.dodaj_ciecz(100 - self.zp2.aktualna_ilosc); self.update()
    def odejmij_zp2(self): self.zp2.dodaj_ciecz(-self.zp2.aktualna_ilosc); self.update()

    def __init__(self):
        super().__init__()
        self.setWindowTitle("Wizualizacja")
        self.setFixedSize(1300, 650) 
        self.setStyleSheet("background-color: #222;")

        self.temperatura = 50
        self.btn_grzania_ref = None
        self.timer_grzania = QTimer()
        self.timer_grzania.timeout.connect(self.krok_symulacji_grzania)

        self.pa1 = para(660, 250, 80, 60)
        self.pary = [self.pa1]
        self.g1 = grzanie(690, 510, width=50, height=20, nazwa="Grzanie")
        self.grzanie = [self.g1]
        self.t1 = pompa(250, 125-7, nazwa="Pompa 1")
        self.t2 = pompa(525, 275-7, nazwa="Pompa 2")
        self.pompy = [self.t1, self.t2]

        self.z1 = Zbiornik(50, 50, nazwa="Zbiornik 1")
        self.z1.aktualna_ilosc = 100.0; self.z1.aktualizuj_poziom()
        self.z2 = Zbiornik(350, 200, nazwa="Zbiornik 2")
        self.z3 = Zbiornik(650, 350, nazwa="Zbiornik 3")
        self.zp1 = Zbiornik_para(1100, 50, nazwa="Zbiornik Para 1")
        self.zp2 = Zbiornik(1100, 400, nazwa= "Zbiornik Para 2")
        self.zbiorniki = [self.z1, self.z2, self.z3, self.zp2]
        self.zbiorniki_para = [self.zp1]

        kolor_woda = QColor(0, 180, 255)
        kolor_para = QColor(220, 220, 220)

        # Rury z wodą
        p1s = self.z1.punkt_dol_srodek(); p1e = self.z2.punkt_gora_srodek(); m1y = (p1s[1]+p1e[1])/2
        self.rura1 = Rura([p1s, (p1s[0], m1y), (p1e[0], m1y), p1e], kolor_cieczy=kolor_woda); self.t1.y = m1y - 7

        p2s = self.z2.punkt_dol_srodek(); p2e = self.z3.punkt_gora_srodek(); m2y = (p2s[1]+p2e[1])/2
        self.rura2 = Rura([p2s, (p2s[0], m2y), (p2e[0], m2y), p2e], kolor_cieczy=kolor_woda); self.t2.y = m2y - 7

        # Rury z parą
        p3s = self.z3.punkt_gora_srodek(); h_mag = 280; x_split = 1000
        self.rura3 = Rura([p3s, (p3s[0], h_mag), (x_split, h_mag)], kolor_cieczy=kolor_para)
        
        h_wlot1 = self.zp1.y + 70
        self.rura_do_zp1 = Rura([(x_split, h_mag), (x_split, h_wlot1), (self.zp1.x, h_wlot1)], kolor_cieczy=kolor_para)
        
        h_wlot2 = self.zp2.y + 50
        self.rura_do_zp2 = Rura([(x_split, h_mag), (x_split, h_wlot2), (self.zp2.x, h_wlot2)], kolor_cieczy=kolor_para)

        self.rury = [self.rura1, self.rura2, self.rura3, self.rura_do_zp1, self.rura_do_zp2]

        self.timer = QTimer()
        self.timer.timeout.connect(self.logika_przeplywu)
        self.running = False; self.flow_speed = 0.4; self.on = False

    #Logika grzałek
    
    def podgrzewanie(self, btn_ref=None):
        self.btn_grzania_ref = btn_ref
        
        if not self.g1.czy_grzeje:
            if self.z3.aktualna_ilosc <= 5:
                print("BŁĄD: Za mało wody w Zbiorniku 3!")
                return
            self.g1.czy_grzeje = True
            self.temperatura = 50 
            if btn_ref: 
                btn_ref.setStyleSheet("background-color: green; color: white; font-weight: bold; padding: 10px;")
            self.timer_grzania.start(200) 
            print("Grzanie włączone.")
        else:
            self.wylacz_grzanie()
            print("Grzanie wyłączone ręcznie.")

    def wylacz_grzanie(self):
        self.g1.czy_grzeje = False
        self.timer_grzania.stop()
        for pa in self.pary: pa.widoczna = False
        if self.btn_grzania_ref:
            self.btn_grzania_ref.setStyleSheet("background-color: red; color: white; font-weight: bold; padding: 10px;")
        self.update()

    def krok_symulacji_grzania(self):
        self.temperatura += 1
        print(f"Temperatura wody: {self.temperatura} C")

        if self.temperatura >= 96:
            print("Awaria: Przegrzanie!")
            self.wylacz_grzanie(); return

        if self.z3.aktualna_ilosc <= 5:
            print("Awaria: Woda się skończyła!")
            self.wylacz_grzanie(); return
            
        if self.zp1.czy_pelny() and self.zp2.czy_pelny():
            print("Awaria: Zbiorniki pary pełne!")
            self.wylacz_grzanie(); return

        if 85 <= self.temperatura <= 95:
            for pa in self.pary: pa.widoczna = True
            ilosc = 4.0
            self.z3.usun_ciecz(ilosc)
            if not self.zp1.czy_pelny() and not self.zp2.czy_pelny():
                self.zp1.dodaj_ciecz(2); self.zp2.dodaj_ciecz(2)
            elif self.zp1.czy_pelny(): self.zp2.dodaj_ciecz(4)
            elif self.zp2.czy_pelny(): self.zp1.dodaj_ciecz(4)
        else:
            for pa in self.pary: pa.widoczna = False
        self.update()

    # Logika przepływów i raportu

    def praca_pomp(self):
        self.on = not self.on
        self.flow_speed = self.flow_speed * 2 if self.on else self.flow_speed / 2
        print(f"Pompy: {'ON' if self.on else 'OFF'}")

    def przelacz_symulacje(self):
        if self.running: 
            self.timer.stop()
            print("\n" + "="*30)
            print(" ZATRZYMANO SYMULACJĘ - RAPORT")
            print("="*30)
            print(f" Zbiornik 1:      {self.z1.aktualna_ilosc:6.1f} / 100.0")
            print(f" Zbiornik 2:      {self.z2.aktualna_ilosc:6.1f} / 100.0")
            print(f" Zbiornik 3:      {self.z3.aktualna_ilosc:6.1f} / 100.0")
            print(f" Zbiornik Para 1: {self.zp1.aktualna_ilosc:6.1f} / 100.0")
            print(f" Zbiornik Para 2: {self.zp2.aktualna_ilosc:6.1f} / 100.0")
            print("="*30 + "\n")
        else: 
            self.timer.start(20)
            print("Uruchomiono przepływ.")
        self.running = not self.running

    def logika_przeplywu(self):
        plynie_1 = False
        if not self.z1.czy_pusty() and not self.z2.czy_pelny():
            self.z2.dodaj_ciecz(self.z1.usun_ciecz(self.flow_speed))
            plynie_1 = True
        self.rura1.ustaw_przeplyw(plynie_1)
        
        plynie_2 = False
        if (self.z2.aktualna_ilosc > 15 or (self.z2.aktualna_ilosc > 0 and not plynie_1)) and not self.z3.czy_pelny():
             self.z3.dodaj_ciecz(self.z2.usun_ciecz(self.flow_speed))
             plynie_2 = True
        self.rura2.ustaw_przeplyw(plynie_2)
        
        plynie_para = self.g1.czy_grzeje and any(p.widoczna for p in self.pary)
        self.rura3.ustaw_przeplyw(plynie_para)
        self.rura_do_zp1.ustaw_przeplyw(plynie_para)
        self.rura_do_zp2.ustaw_przeplyw(plynie_para)
        
        self.update()

    def paintEvent(self, event):
        p = QPainter(self); p.setRenderHint(QPainter.Antialiasing)
        for r in self.rury: r.draw(p)
        for z in self.zbiorniki: z.draw(p)
        for zp in self.zbiorniki_para: zp.draw(p)
        for t in self.pompy: t.draw(p)
        for g in self.grzanie: g.draw(p)
        for pa in self.pary: pa.draw(p)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    okno_symulacji = SymulacjaKaskady()
    okno_symulacji.show()
    panel_sterowania = PanelSterowania(okno_symulacji)
    panel_sterowania.show()
    sys.exit(app.exec_())
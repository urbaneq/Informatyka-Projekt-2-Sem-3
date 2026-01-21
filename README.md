# Informatyka-Projekt-2-Sem-3
Program napisany w języku Python przy użyciu biblioteki PyQt5. Aplikacja wizualizuje proces przepływu cieczy między zbiornikami, pracę pomp oraz proces generowania i dystrybucji pary wodnej.
Do uruchomienia programu potrzebny jest zainstalowany Python (wersja 3.x) oraz biblioteka PyQt5.
Funkcjonalności:
- Symulacja przepływu: Woda przepływa grawitacyjnie ze zbiornika górnego do dolnego.
- System Pomp: Możliwość włączenia pomp, które przyspieszają przepływ wody (wizualizowane kolorem zielonym).
- System Grzewczy: Symulacja grzałki (kształt zygzaka), która podgrzewa wodę w ostatnim zbiorniku.
- Generowanie Pary: Po osiągnięciu odpowiedniej temperatury (85°C - 95°C), woda zamienia się w parę i jest transportowana szarymi rurami do zbiorników pary.
- Panel Sterowania: Osobne okno z przyciskami do pełnej kontroli nad systemem.
- Raportowanie: Po zatrzymaniu symulacji, w konsoli wypisywany jest dokładny raport o stanie napełnienia wszystkich zbiorników.
- Zabezpieczenia: System automatycznie wyłącza grzanie w przypadku braku wody, przepełnienia zbiorników pary lub przegrzania.

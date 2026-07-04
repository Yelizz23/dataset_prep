# Dataset pro fine-tuning LLM: Swift - Investigace
## Domací úkol - kurz AI Architect od robot_dreams

## 1. Cil datasetu

Cilem je naucit velky jazykovy model (LLM) **nove domenove znalosti**, ktere nejsou v jeho trenovacich datech: transformaci procesu vyjimek a investigaci (Exceptions & Investigations, E&I) a ruseni plateb v ramci migrace na ISO 20022, vcetne reseni **Swift Case Management** (Case Orchestrator a Stop and Recall) a zavaznych terminu **listopad 2026** a **listopad 2027**.

Zvoleny zpusob zmeny chovani modelu: **supervised fine-tuning (SFT)** pomoci instrukcnich dvojic otazka-odpoved v konverzacnim (chat) formatu. Model se tak nauci fakticky spravne odpovidat na dotazy bank, produktovych manazeru a IT tymu ohledne migrace E&I.

## 2. Zdroj dat

- Primarni zdroj: verejny clanek Swift *"Transforming exceptions and investigations"* (publikovano 7. 5. 2026)
  https://www.swift.com/news-events/news/transforming-exceptions-and-investigations
- Z clanku byla extrahovana pouze faktograficka jadra (terminy, nazvy sluzeb, typy zprav, pravidla, doporuceni). Vsechny odpovedi v datasetu jsou parafrazovane.

## 3. Postup sberu a cisteni dat

1. **Sber**: Extrakce hlavniho textu clanku.
2. **Cisteni**:
   - normalizace textu (sjednoceni pomlcek, uvozovek, mezer).
3. **Extrakce faktu**: rucni identifikace klicovych faktu a jejich overeni proti textu (kazdy fakt v datasetu je dolozitelny zdrojovym clankem):
   - listopad 2025 = konec koexistence MT/ISO 20022 pro platebni instrukce (CBPR+),
   - Case Management = Case Orchestrator (camt.110/111) + Stop and Recall (camt.056/029, blokovani UETR),
   - 50 % cilovych instituci live nebo naplanovano do listopadu 2026,
   - listopad 2026 = povinny prijem camt.110 (vlozena MT 199), in-flow translace,
   - listopad 2027 = konec in-flow translace, povinne ISO zpravy, konec MT pro ruseni plateb, pouze FINplus,
   - camt.110 zdarma, in-flow translace platebnich instrukci zpoplatnena,
   - testovani v Test Sparring Partner v prubehu 2026,
   - dve strategie pripravy: early adoption vs. mass enablement.
4. **Strukturovani**: tvorba dvojic otazka-odpoved pokryvajicich fakta z ruznych uhlu (definice, terminy, srovnani, casova osa, prakticke scenare, role-play dotazy  pracovniku backoffice). Cca 20 % prikladu je v anglictine pro robustnost modelu vuci dotazovacimu jazyku.
5. **Kontrola kvality**:
   - kazda odpoved overena proti zdroji (zadne halucinace),
   - deduplikace otazek,
   - validace JSONL syntaxe (kazdy radek = validni JSON objekt),
   - konzistentni system prompt v ramci jazyka.

## 4. Struktura datasetu

Format: **JSONL** (jeden trenovaci priklad na radek), konverzacni schema kompatibilni s beznymi fine-tuning API:

```json
{"messages": [
  {"role": "system", "content": "Jsi odborny asistent na platebni infrastrukturu Swift..."},
  {"role": "user", "content": "Otazka uzivatele"},
  {"role": "assistant", "content": "Fakticky spravna odpoved"}
]}
```

| Soubor            | Pocet prikladu | Ucel                            |
|-------------------|----------------|---------------------------------|
| `train.jsonl`     | 37             | trenovaci sada                  |
| `validation.jsonl`| 7              | validacni sada (drzena stranou) |
| `test.jsonl`      | 5              | testovaci sada (drzena stranou) |

Rozdeleni train/validation je pribl. 84/16 %. Validacni priklady pokryvaji stejna fakta jinymi formulacemi, aby merily generalizaci, ne memorovani formulaci.

## 5. Typy prikladu (pokryti)

- **Definicni**: Co je Case Management, UETR, E&I, camt.110/111.
- **Faktograficke/terminove**: milniky 2025, 2026, 2027; podil zapojenych instituci.
- **Srovnavaci**: Case Orchestrator vs. Stop and Recall; early adoption vs. mass enablement.
- **Procesni**: jak funguje in-flow translace, zpoplatneni, testovani.
- **Aplikacni (role-play)**: dotazy z pohledu banky, produktoveho manazera - overuji, ze model umi znalosti pouzit, ne jen recitovat.
- **Sumarizacni**: casova osa, seznam zdroju.
- **Dvoujazycne**: cestina (bez diakritiky kvuli technickym pozadavkum odevzdani) + anglictina.


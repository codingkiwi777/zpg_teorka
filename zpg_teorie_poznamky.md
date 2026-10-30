# 📒 ZPG – teorie z přednášek 1–6 (moje poznámky, doplněné)

✏️ = opravené nebo upřesněné oproti mým původním poznámkám · 🆕 = nově doplněné · číslo v závorce = přednáška

---

## 📋 0) Organizace (P1)
- **HODNOCENÍ**
  - zápočet **45 b** (min. 20): projekt v C++ a OpenGL + obhajoba v zápočtovém týdnu
  - zkouška **55 b**: písemka **25 b** (min. 10, zápočtový týden na přednášce) + ústní **30 b** (min. 10, 2 otázky)
  - AI se smí použít jen na věci, které nesouvisí s probíranou látkou (např. generování modelů)
- **STÁTNICE** (co se z toho bude chtít): zobrazovací řetězec, modelovací a zobrazovací transformace, **Phongův model**, viditelnost, afinní a projektivní prostory, reprezentace těles, OpenGL a GLSL

---

## 🖥️ 1) Grafika, API a OpenGL (P1, P2)

- 🆕 **DĚLENÍ POČÍTAČOVÉ GRAFIKY**
  - podle dimenze: **2D / 3D**
  - podle popisu: **rastrová / vektorová**
  - podle rychlosti: **real-time** (hry, náš projekt) / **fotorealistický** rendering (ray tracing, předmět PG1)
- 🆕 **RASTR vs. VEKTOR**
  - 🟦 rastr = obrázek z **pixelů**, každý má svou barvu, při zvětšení nebo rotaci **ztrácí kvalitu**, hodí se na fotky
  - 📐 vektor = obraz z **matematických objektů** (bod, úsečka, kružnice, křivka), zvětšení **bez ztráty kvality**, nehodí se na realistické fotky
  - **RASTERIZACE** = převod vektorového popisu na rastr (mřížku), rastr musí mít danou velikost
  - 🔤 fonty: **TrueType** = kvadratické Bézierovy křivky (menší) · **PostScript** = kubické Bézierovy křivky (přesnější, pro tisk)
- 🆕 **GRAFICKÁ API**
  - **OpenGL** (Khronos) = vyšší abstrakce, jednodušší, trojúhelník ≈ 100 řádků
  - **Vulkan** (Khronos, nástupce OpenGL, 2016) = nízkoúrovňové, plná kontrola, více vláken, trojúhelník ≈ 1000 řádků, shadery ve formátu **SPIR-V**
  - **DirectX 12** (Microsoft) = nízkoúrovňové, Windows a Xbox
- 🆕 **OPENGL**
  - multiplatformní API jen na **vykreslování** (okna, klávesnici a myš neřeší → proto GLFW)
  - historie: 1992 OpenGL 1.0 (Silicon Graphics) · 2003 1.5 VBO · 2004 **2.0 GLSL a shadery** · 2006 přechod pod Khronos · 2008 **3.0 VAO**, 3.1 odstraňuje fixní pipeline · 2009 3.2 geometry shader · 2010 4.0 teselace · 2017 **4.6** (aktuální)
  - cesta příkazu: **Program → OpenGL API → driver GPU → grafická karta**. Driver dělá výrobce (NVIDIA, AMD, Intel), proto stejný program běží na různých kartách.
  - **GLSL** = jazyk pro psaní shaderů
- 🆕 **FIXNÍ vs. PROGRAMOVATELNÁ PIPELINE**
  - 🔒 fixní (OpenGL 1.x, `glBegin/glEnd`, `glTranslatef`, `glRotatef`) = pevné kroky, jde jen **nastavovat**, ne programovat
  - 🔓 programovatelná (od 2.0) = kroky si píšu sám jako **shadery** (program běžící na GPU)
  - **Core profile** = bez starých funkcí (my: 3.3 core) · **Compatibility profile** = zpětně kompatibilní
  - (ne)unifikované shadery: dřív pevný počet jednotek pro vertex a fragment, dnes **unifikované** = GPU si typ jednotky mění podle potřeby
- 🆕 **PER-VERTEX vs. PER-FRAGMENT OPERACE**
  - per-vertex (každý vrchol zvlášť): transformace pozice, normál, příprava hodnot k interpolaci
  - per-fragment (každý fragment z rasterizace): výsledná barva, textury, osvětlení, depth test, blending

---

## 🚂 2) Vykreslovací řetězec a shadery (P1, P2)

- 🆕 **VYKRESLOVACÍ ŘETĚZEC (pipeline)**
  - **Vertex shader** → transformace vrcholů, výsledek do `gl_Position`
  - **Clipping** → ořezání toho, co není vidět
  - **Rasterizace** → rozklad trojúhelníků na **fragmenty** (vektor → rastr)
  - **Fragment shader** → barva (a hloubka) fragmentu
  - → framebuffer (obrazovka)
- 🆕 **VERTEX SHADER**
  - běží **pro každý vrchol**, mění jeho polohu
  - **nemůže přidávat ani mazat vrcholy**, nezná sousední vrcholy (neví nic o tělese)
  - pracuje v homogenních souřadnicích → `vec4(position, 1.0)`
- 🆕 **FRAGMENT SHADER**
  - běží **pro každý fragment**, počítá jen **barvu** (osvětlení, textury, mlha)
  - **nemůže změnit pozici** fragmentu
  - fragment ≠ pixel: pixel se může skládat z více fragmentů (anti-aliasing)
- 🆕 **GEOMETRY SHADER (3.2)** = může vrcholy **přidávat, měnit, mazat**, zná sousední vrcholy (`EmitVertex`, `EndPrimitive`)
- 🆕 **TESELAČNÍ SHADER (4.0)** = rozdělí polygony na menší → zaoblení tvaru (Bézierovy pláty), nezaměňovat s displacement mappingem
- 🆕 **PROMĚNNÉ V SHADERECH**
  - `in` (z VBO) = každý vrchol jiná hodnota
  - `uniform` (z C++) = stejné pro všechny vrcholy jednoho vykreslení, program si hodnotu **pamatuje**
  - `out` ve vertexu → `in` ve fragmentu = předání dál, hodnota se mezi vrcholy **interpoluje**
- 🆕 **SHADER PROGRAM (postup)**
  - `glCreateShader` → `glShaderSource` → `glCompileShader` (zvlášť vertex a fragment)
  - `glCreateProgram` → `glAttachShader` ×2 → `glLinkProgram`
  - kontrola: `glGetShaderiv(GL_COMPILE_STATUS)` + `glGetShaderInfoLog`, při chybě konec aplikace
- 🆕 **UNIFORM Z C++**
  - `glGetUniformLocation(program, "jméno")` → číslo pozice, **−1 = proměnná neexistuje** (nebo ji kompilátor vyhodil, protože se nepoužívá)
  - chyby: `GL_INVALID_OPERATION`, když program není slinkovaný nebo to není program
  - `glUniformMatrix4fv(location, 1, GL_FALSE, value_ptr(M))` = (pozice, počet, transponovat?, data)
  - před tím musí být `glUseProgram`

- **clock.vert** ✏️
  - shader podle **barvy vrcholu** pozná ručičku, z času (uniform `clockTime`) spočítá její úhel, otočí vrchol kolem středu hodin a pak ho běžně promítne na obrazovku pomocí **P·V·M**
  - 🆕 proč tak: celé hodiny = **1 draw call** (vertex colors), ručičky otáčí GPU, ne CPU

---

## 🧱 3) Data modelů na GPU (P2, P4)

- 🆕 **ZPŮSOBY KRESLENÍ (historie)**
  - `glBegin/glEnd` = data se posílají **každý snímek znovu** → pomalé (deprecated od 3.1)
  - Display list = objekt „nahraný“ jedním příkazem, ale při změně nový list (deprecated)
  - Vertex Arrays = pole, ale při každém kreslení se musí poslat na GPU (deprecated)
  - ✅ **VBO + VAO** = dnešní způsob
- 🆕 **VBO (Vertex Buffer Object)**
  - kus paměti **přímo na grafické kartě** s daty vrcholů (pozice, normály, barvy, uv)
  - `glGenBuffers` → `glBindBuffer(GL_ARRAY_BUFFER)` → `glBufferData(…, GL_STATIC_DRAW)`
- 🆕 **VAO (Vertex Array Object)**
  - **návod, jak číst VBO** (co je pozice, co normála), může obsahovat více VBO
  - `glVertexAttribPointer(index, počet, typ, normalized, stride, offset)`
    - např. pozice + normála: stride = `6 * sizeof(float)`, normála má offset `3 * sizeof(float)`
  - `glEnableVertexAttribArray(i)` pro každý atribut
- 🆕 **IBO / EBO (index buffer)** = indexy vrcholů trojúhelníků, když se vrcholy opakují
  - bez indexů `glDrawArrays` · s indexy `glDrawElements`
- 🆕 **VERTEX COLORS** = barva uložená přímo ve vrcholu (bez textury), dá se použít i na **přídavná data** pro shader (u hodin: která ručička)
- 🆕 **DRAW CALL**
  - příkaz CPU → GPU „vykresli“ (`glDrawArrays`, `glDrawElements`)
  - každý má **režii na CPU** → čím méně, tím lépe (100 objektů = 100 draw callů, sloučená geometrie = 1)
- 🆕 **Z-BUFFER** = `glEnable(GL_DEPTH_TEST)` + mazat `GL_DEPTH_BUFFER_BIT`, jinak se zadní trojúhelníky kreslí přes přední
- 🆕 **VYKRESLOVACÍ SMYČKA**: `glClear` → `glUseProgram` → `glBindVertexArray` → `glDrawArrays` → `glfwPollEvents` → `glfwSwapBuffers`

---

## 📐 4) Prostory a základní pojmy (P1, P3)

- **EUKLIDOVSKÝ PROSTOR** ✏️
  - prostor, ve kterém umíme měřit **délky a úhly**
  - dvojice **(E, ⟨·,·⟩)** = reálný vektorový prostor + **skalární součin**
  - ✏️ má **přirozený počátek** (všechno jsou vektory z nuly)
  - 🆕 v grafice = **model** (tvary, délky, úhly)
 ↓
- **AFINNÍ PROSTOR**
  - je prostor, ve kterém rozlišuju **body** (místa) a **vektory** (posuny). Bod + vektor = nový bod, bod − bod = vektor, sčítat dva body nejde.
  - ✏️ dvojice **(A, E)** = množina bodů + přidružený vektorový prostor (směry)
  - neexistuje přirozený počátek – musíme ho **zvolit** (bod O, pak každý bod P = vektor P − O → chová se jako euklidovský)
  - 🆕 v grafice = **scéna** (posun, rotace, měřítko)
 ↓
- **PROJEKTIVNÍ PROSTOR** ✏️
  - 🆕 rozšiřuje afinní prostor o **body v nekonečnu** → umí **perspektivu** (rovnoběžky se protnou v nekonečnu)
  - bod je o **jednu dimenzi víc** (3D → 4D) = **homogenní souřadnice** `(x·w, y·w, z·w, w)`, w ≠ 0
  - zpět do afinního = **vydělit w**: `(x/w, y/w, z/w)`
  - díky tomu jde **i posun zapsat maticí** → všechny transformace jsou matice **4×4** a dají se násobením složit do jedné ✅ (= hlavní výhoda)
  - nejčastěji **w = 1** (bod); ✏️ w = 0 = **bod v nekonečnu = směr** → proto vektor (normálu) píšu s w = 0 a posun na něj nepůsobí
  - 🆕 v grafice = **obrazovka** (projekce)
  - 🆕 stejný bod, jiné w: A[2,2] → (2,2,1) = (4,4,2) = (20,20,10). Když w → 0, bod utíká do nekonečna.
- 🆕 **Euklidovský ⊂ Afinní ⊂ Projektivní**

| | Euklid. | Afinní | Projekt. |
|---|---|---|---|
| skalární součin, délky, úhly | ✅ | ❌ | ❌ |
| rovnoběžnost | ✅ | ✅ | ❌ |
| posun, rotace, měřítko | ✅ (měřítko omezeně) | ✅ | ✅ |
| zkosení (shear) | ❌ | ✅ | ✅ |
| perspektiva, body v ∞ | ❌ | ❌ | ✅ |

- 🆕 **SKALÁR** = veličina jen s velikostí (hmotnost, teplota)
- **BOD** ✏️
  - základní **bezrozměrný** útvar, v prostoru trojice reálných čísel **[x, y, z]**
  - 🆕 v jiné souřadné soustavě má tentýž bod jiné souřadnice
- **VEKTOR** ✏️
  - prvek vektorového prostoru, má **velikost a směr**, 🆕 **nemá pevnou polohu**
  - 🆕 popisuje **posunutí** z bodu do bodu: **u = B − A** („jdi 10 m na jih“)
  - velikost (norma) = odmocnina ze **součtu druhých mocnin** složek: `|v| = √(v1² + v2² + v3²)`
- **SKALÁRNÍ SOUČIN (dot)**
  - umožňuje definovat **DÉLKY, ÚHLY, VZDÁLENOSTI A KOLMOST**
  - jak moc míří dva vektory stejným směrem
  - výsledek je skalár (číslo) – součet součinů jednotlivých složek: `x1·y1 + x2·y2 + x3·y3`
  - geometricky je to délka krát délka krát kosinus úhlu, takže u normalizovaných vektorů přímo kosinus: `⟨x,y⟩ = |x|·|y|·cos θ`
  - 🆕 vzorce:
    - délka: `|x| = √⟨x,x⟩` (vektor sám se sebou = délka²)
    - úhel: `cos α = ⟨x,y⟩ / (|x|·|y|)`
    - vzdálenost bodů: `d(P,Q) = |Q − P|`
    - kolmé ⇔ `dot = 0`
  - 🆕 příklad: u = (3,0), v = (2,2) → dot = 6, |u| = 3, |v| = √8 → cos α = 1/√2 → **α = 45°**
  - 3 pravidla (musí splňovat):
    - **symetrie** – nezáleží na pořadí
    - **linearita** ✏️ (v prvním argumentu) – dvakrát delší šipka → dvakrát větší dot, a `⟨x+y, z⟩ = ⟨x,z⟩ + ⟨y,z⟩`
    - **pozitivní definitnost** – šipka sama se sebou dá vždycky číslo ≥ 0 (svoji délku na druhou) ✏️ a **0 jen pro nulový vektor**
- **VEKTOROVÝ SOUČIN (cross)** ✏️
  - vezmeš dvě šipky a dostaneš třetí šipku, která je **kolmá na obě**
  - 🆕 směr = **pravidlo pravé ruky** (ukazováček a, prostředníček b, palec a × b) → **záleží na pořadí** (b × a míří opačně)
  - 🆕 velikost = **obsah rovnoběžníku** = `|a|·|b|·sin θ` → rovnoběžné vektory dají **nulu**
  - 🆕 vzorec: `a × b = (a2b3 − a3b2, a3b1 − a1b3, a1b2 − a2b1)`
  - 🆕 v projektu: kamera A/D `cross(target, up)` = doprava · normála trojúhelníku · uvnitř `lookAt`
- **NORMÁLA**
  - šipka **kolmo na povrch** (míří ven), říká, jak je plocha natočená; ✏️ pro výpočty ji vždy **normalizujeme** na délku 1
  - SPOČÍTÁME ji vektorovým součinem (dvě hrany trojúhelníku): `N = normalize((B − A) × (C − A))`
- **PŘÍMKA** ✏️
  - `X(t) = A + u·t, t ∈ R` (parametrické vyjádření, A = bod, u = směrový vektor)
  - 🆕 přímka dvěma body: `X(t) = A + (B − A)·t`
  - 🆕 tvar pro **interpolaci**: `X(t) = (1 − t)·A + t·B`
  - 🆕 **úsečka** = stejné, ale `t ∈ ⟨0, 1⟩` (t = 0 → A, t = 1 → B)
  - 🆕 **polopřímka** = `t ≥ 0` (začne v A a jde ve směru u) → takhle vypadá i promítací paprsek
- **NORMALIZE(v)** = normalizovaný (jednotkový) vektor
  - vezme vektor a zkrátí nebo prodlouží ho na **délku 1**: `v / |v|`, směr zůstane
- **ÚHLY**
  - cos je vodorovná (x) a sin svislá (y) souřadnice bodu na **jednotkové kružnici**: `A = [cos α, sin α]`
  - 🆕 v trojúhelníku: sin = protilehlá / přepona, cos = přilehlá / přepona
  - 🆕 `cos(x) = sin(x + π/2)`, obor hodnot ⟨−1, 1⟩
  - C++, GLM i GLSL počítají v **radiánech** (`glm::radians(°)`)
- **TEČNA A NORMÁLA** ✏️
  - tečna leží **na ploše** a normála je na ni kolmá → `dot(tečna, normála) = 0`
  - 🆕 tečný vektor = rozdíl dvou vrcholů: `t = V2 − V1`

---

## 🔄 5) Afinní transformace (P3)

- 🆕 **AFINNÍ TRANSFORMACE** = každému bodu A přiřadí obraz A′
  - **zachová**: přímky zůstanou přímkami · rovnoběžky rovnoběžkami · **poměry na přímce** (střed úsečky zůstane středem)
  - **nemusí zachovat**: délky a úhly
  - obecně: `X′ = A·X + d` (A = matice rotace/měřítka, d = posun)
- **TRANSLACE (posun)** = přičtu vektor posunu
  - `x′ = x + dx, y′ = y + dy` → `A′ = A + d`
  - ⚠️ 3×3 maticí to nejde (je to sčítání) → proto 4×4 a w = 1:
  ```
  T(a,b,c) = | 1 0 0 a |
             | 0 1 0 b |
             | 0 0 1 c |
             | 0 0 0 1 |
  ```
- **ROTACE** = pohyb bodu po **kružnici kolem počátku** o úhel α
  - **CCW** (proti hodinkám) = standard: `x′ = x·cos α − y·sin α`, `y′ = x·sin α + y·cos α`
  - matice CCW `[cos −sin; sin cos]`, CW `[cos sin; −sin cos]`
  - 🆕 odvození: bod = `[r·cos α, r·sin α]`, po otočení o β = `[r·cos(α+β), r·sin(α+β)]` + součtové vzorce
  - 🆕 jde chápat jako otočení **bodu**, nebo otočení **souřadné soustavy**
  - 🆕 3D: kolem osy x (`Rx`, mění se y a z), y (`Ry`), z (`Rz`, mění se x a y)
  ```
  Rz(α) = | cos −sin 0 0 |
          | sin  cos 0 0 |
          |  0    0  1 0 |
          |  0    0  0 1 |
  ```
- **SCALE (změna měřítka)** = vynásobím každou souřadnici svým měřítkem, **vzhledem k počátku**
  - `x′ = sx·x, y′ = sy·y` → matice s `sx, sy, sz` na diagonále
  - 🆕 uniformní (všude stejné) vs. **neuniformní** (různé na osách → problém s normálami)
- 🆕 **POŘADÍ (násobení matic NENÍ komutativní)**
  - čte se **zprava doleva**: `P·V·M·v` = nejdřív M
  - A[1,1]: rotace 90° a pak posun (1,0) → **[0, 1]** · posun a pak rotace → **[−1, 2]**
  - v projektu: `T·R` = točí se na místě, `R·T` = obíhá
- 🆕 **ROTACE KOLEM OBECNÉHO BODU S**
  - posunu S do počátku → otočím → posunu zpět: **`M = T(a,b) · R(α) · T(−a,−b)`**
  - příklad: A = [5, 4] kolem C = [2, 3] o 90° → **A′ = [1, 6]**
  - 🆕 kolem obecné **osy** ve 3D: `M = T2 · R5 · R4 · R3 · R2 · R1 · T1` (osu posunu a natočím do osy souřadnic, otočím, vrátím)
- 🆕 **SKLÁDÁNÍ**: `Xn = An·(…A2·(A1·X + d1) + d2…) + dn` → v homogenních souřadnicích prostě **jedna matice** M = Mn·…·M1
- 🆕 **SLOUPCOVÝ vs. ŘÁDKOVÝ VEKTOR**: `X′ = A·X` vs. `X′ = X·B`, kde **B = Aᵀ** (transpozice). GLSL/GLM = sloupcový → `M * vec4(p, 1)`.
- 🆕 **ZA SNÍMEK (P4)**
  - `alfa += 0.5; M = rotate(mat4(1), alfa, …)` = matice se pokaždé počítá **znovu od jednotkové** s celkovým úhlem
  - `M = rotate(M, 0.5, …)` = otočení se **přičítá k minulé matici** (hromadí se, i chyby zaokrouhlení)
- 🆕 **GLM**: `glm::mat4(1.0f)` = jednotková · `translate`, `rotate` (radiány!), `scale`

---

## 📷 6) Souřadné systémy, kamera, projekce (P3, P4)

- 🆕 **CESTA VRCHOLU (souřadné systémy)**
  - **Object/Local space** –(M modelová)→ **World space** –(V pohledová)→ **View/Camera space** –(P projekční)→ **Clip space** –(viewport)→ **Screen space**
  - lokální = souřadnice modelu (střed modelu) · globální = rozmístění modelů ve scéně
  - `gl_Position = P · V · M · vec4(pozice, 1.0)`
- 🆕 **ROTACE POMOCÍ BÁZOVÝCH VEKTORŮ**
  - 3 navzájem kolmé jednotkové vektory a, b, c (**ortonormální báze**) dám do **sloupců** → rotační matice R
  - **otočit bod**: `A′ = R·A` (bod se pohne)
  - **souřadnice pevného bodu v nové bázi**: `A′ = Rᵀ·A` (bod stojí, mění se jen popis)
  - rotační matice je **ortogonální** → **R⁻¹ = Rᵀ** (inverze = transpozice, levné)
  - příklad 90° kolem z: A = [1,1,0] → otočený bod [−1, 1, 0], v nové bázi [1, −1, 0]
- 🆕 **POHLEDOVÁ TRANSFORMACE (kamera, V)**
  - výchozí kamera: v **počátku**, kouká po **−z**, nahoru je **+y**
  - `lookAt(eye, center, up)`: **eye** = kde stojím · **center** = kam koukám (bod) · **up** = kde je nahoře
  - uvnitř: `v = normalize(center − eye)` (dopředu) · `r = normalize(v × up)` (doprava) · `u = normalize(r × v)` (nahoru)
  - `view = A · B`: **B = posun o −eye** (svět posunu tak, aby kamera byla v počátku), **A = rotace** s řádky r, u, −v
  - 💡 kamera se nehýbe, **hýbe se celý svět opačně**
  - příklad: eye (1,0,0), center 0, up y → v = (−1,0,0), r = (0,0,−1), u = (0,1,0)
- 🆕 **SFÉRICKÉ SOUŘADNICE** (kam kamera kouká)
  - 2D polární: `x = r·cos φ, y = r·sin φ`
  - 3D: `x = r·sin α·cos φ`, `y = r·cos α`, `z = r·sin α·sin φ`, α ∈ ⟨0°, 180°⟩, φ ∈ ⟨0°, 360°⟩
  - u nás target, start α = 90°, φ = 270° → (0, 0, −1)
- 🆕 **POHYB KAMERY**: W = `eye += target`, A/D = `eye ∓ normalize(cross(target, up))` (na slajdu je u toLeft chybně `+=`)
- 🆕 **PROMÍTÁNÍ** = převod **3D → 2D**
  - **promítací paprsek** = polopřímka z promítacího bodu · **průmětna** = plocha, kam paprsky dopadají (nemusí být rovina)
  - v geometrii volím nejdřív metodu, pak promítám; v grafice obráceně
  - **rovnoběžné (ortografické)** – pravoúhlé nebo kosoúhlé; scéna = **pravoúhlý hranol**; rovnoběžky zůstanou rovnoběžné; `glm::ortho(left, right, bottom, top, near, far)`
  - **středové (perspektivní)** – scéna = **komolý jehlan (frustum)**; vzdálené věci menší; `glm::perspective(fovy, aspect, near, far)` nebo `glm::frustum(…)`
  - perspektiva z **podobnosti trojúhelníků**: `x′ = k·x`, `k = d / (d + y)` → dělení vzdáleností = proto matice zapíše hodnotu do **w** a GPU pak **dělí w**
  - parametry: **FOV** (úhel pohledu), **aspect** (šířka/výška), **zNear > 0**, **zFar**
- 🆕 **glViewport(x, y, width, height)**
  - převod z **NDC** (−1…1) na **souřadnice okna** (pixely), x, y = levý dolní roh
  - `xw = (x_nd + 1)·width/2 + x` · `yw = (y_nd + 1)·height/2 + y`
  - příklad: A = [−0.5, 1] v (0, 0, 800, 600) → **(200, 600)**
  - `glViewport(50, 50, w − 100, h − 100)` = kreslí se jen do menšího obdélníku s okrajem
  - ⚠️ při změně okna: `glViewport` **+** nový **aspect ratio** v projekci, jinak se tělesa deformují
- 🆕 **GIMBAL LOCK**
  - u **Eulerových úhlů** (rotace postupně kolem x, y, z: `Rz(γ)·Ry(β)·Rx(α)`)
  - po otočení o 90° kolem y se **dvě osy zarovnají** → ztráta **jednoho stupně volnosti**
  - řešení: **kvaterniony**
- 🆕 **KVATERNIONY**
  - Hamilton 1843, rozšíření komplexních čísel: `q = a + bi + cj + dk`, `i² = j² = k² = ijk = −1`
  - 1 reálná + 3 imaginární složky, násobení **není komutativní**
  - rotace o θ kolem jednotkové osy u: `q = (cos θ/2, u·sin θ/2)`, bod: `A′ = q·A·q⁻¹`
- 🆕 **KŘÍŽOVÝ ODKAZ** = dvě třídy na sebe ukazují → v `.h` **dopředná deklarace** `class Camera;`, `#include` až v `.cpp` (u nás ShaderProgram ↔ Camera)

---

## 🧩 7) OOP a návrhové vzory (P2, P3, P5, P6)

- 🆕 **ZÁSADY**
  - každá třída **jedna odpovědnost**, pozor na **„božské“ třídy**, které umí všechno
  - snadno **rozšiřovat** bez změny stávajícího kódu, abstrakce
  - **refaktoring** je normální součást vývoje (UML diagramy)
- 🆕 **TRANSFORMACE V PROJEKTU – 2 přístupy**
  - DrawableObject **je** transformovatelný (dědičnost z TransformableObject)
  - DrawableObject **má** transformaci (agregace/kompozice) ← my
- 🆕 **COMPOSITE** = s jednoduchým (**Leaf**) i složeným (**Composite**) objektem pracuju **stejně** přes společné rozhraní (**Component**), jako soubory a složky
  - Composite má kolekci komponent, `add()`, a `operation()` zavolá na všech dětech
  - u nás: `Transformation` → Translation, Rotation, Scale… + `TransformationComposite` (M = M1·M2·…)
- 🆕 **OBSERVER** = **Subject** informuje **Observery** o změně, aniž je konkrétně zná
  - Subject: `attach`, `detach`, `notifyAll` (projde kolekci a zavolá `notify`)
  - u nás: Camera (Subject) → ShaderProgram (Observer) dostane nové V a P
- 🆕 **DEKLARACE vs. DEFINICE (C++)**
  - deklarace v **.h** = „metoda existuje“ (`void makeSound();`)
  - definice v **.cpp** = tělo metody (`void Cat::makeSound() { … }`)

---

## 💡 8) Osvětlení (P5, P6)

- 🆕 **OSVĚTLOVACÍ MODEL** = barva povrchu podle **světla** (poloha, barva), **materiálu** a **natočení plochy** (normála)
  - modely od nejrychlejšího: Constant → **Lambert → Phong → Blinn** → Cook-Torrance, Strauss, Anisotropic… (kvalita vs. rychlost)
- 🆕 **KONSTANTNÍ SHADER** = jen jedna barva, **ignoruje normály**, všechny plochy stejné (u nás slunce a světluška)
- **AMBIENT** ✏️
  - **intenzita ambientního světla × koeficient (barva) materiálu**: `Ia · ra`
  - 🆕 světlo „odevšad“, zabraňuje tomu, aby odvrácená strana byla **úplně černá**; neutlumuje se
- **LAMBERT (difúzní složka)** ✏️
  - počítá skalární součin (dot) **normály a směru ke světlu**, což je **kosinus úhlu mezi nimi** (oba normalizované), takže plocha natočená ke světlu svítí víc
  - ✏️ (kosinus tu **není** „vodorovná osa“, je to kosinus úhlu mezi n a l: 0° → 1 = plné světlo, 60° → 0.5, 90° → 0)
  - `Id = max(dot(n, l), 0)` → `max` ořízne odvrácenou stranu na 0
  - 🆕 jen ambient + difúzní, **bez lesku** → **matný** povrch
  - 🆕 vzorec: `I_diffuse = Id · rd · max(0, n·l)`
- **lambert.frag** ✏️
  - Pro každý pixel spočítám **směr ke světlu** (`normalize(lightPosition − worldPosition)`) a skalárním součinem s normálou zjistím kosinus úhlu mezi nimi. Ten říká, kolik světla plocha dostane. `max` ořízne odvrácenou stranu na nulu a k tomu přičtu **ambientní složku**, aby stín nebyl úplně černý.
  - ✏️ 10 % (0.1) je příklad ze slajdu; u mě v projektu je ambient z materiálu (`materialAmbient`) a difúzní část je ještě × `materialDiffuse` × `lightColor` × **útlum**
- **PHONG a BLINN** ✏️ = přidávají **zrcadlovou složku (odlesk)**
  - 🆕 **PHONG** (Bui Tuong Phong, **1973**, empirický, ne fyzikální): `I = Ia + Id + Is`
    - odlesk podle úhlu mezi **odraženým paprskem r** a **směrem ke kameře c (V)**
    - `r = reflect(−L, N)` · `spec = pow(max(dot(V, r), 0), h)`
    - `I_specular = Is · rs · max(0, r·c)^h`
  - 🆕 **BLINN-PHONG** (James Blinn, **1977**): místo `reflect` **halfway vektor**
    - `H = normalize(L + V)` (půlka mezi světlem a kamerou) · `spec = pow(max(dot(N, H), 0), h)`
    - levnější, potřebuje **větší h** pro stejný odlesk (u nás 128 vs. 32)
  - 🆕 odlesk má **barvu světla**, ne objektu
- 🆕 **CELÝ PHONGŮV MODEL (pro více světel)**
  - `I = Ia·ra + Σ ( Id,i·rd·(l·n) + Is,i·rs·(r·v)^h )`
- 🆕 **SHININESS (h)** = exponent u `cos^h`: **velké h = malá ostrá tečka** (lesklé), **malé h = široký odlesk** (matnější)
- 🆕 **REFLECT – odvození**
  - GLSL bere vektor **dopadu** `i = −l` (od světla k ploše), proto `reflect(−L, N)`
  - projekce na normálu: `p = (n·i)·n`
  - odraz: **`r = i − 2·(n·i)·n`** (n musí být normalizovaná; když je i jednotkový, je i r jednotkový)
  - příklad: světlo [−1,1,0], F = [0,0,0], n = (0,1,0) → i = (1/√2, −1/√2, 0) → **r = (1/√2, 1/√2, 0)**
- 🆕 **ÚTLUM (attenuation)**
  - fyzika: světlo se ve vakuu neztrácí, ale **rozprostírá na větší plochu** → 2× dál = 4× větší plocha → `1/d²`
  - OpenGL model: **`f_att = 1 / (kc + kl·d + kq·d²)`** (konstantní, lineární, kvadratický koeficient)
  - hodnoty **nejsou dané**, ladí se podle měřítka scény (běžně 1; 0.09; 0.032), výsledek `clamp(…, 0, 1)`
  - násobí difúzní a zrcadlovou složku, ne ambient
- 🆕 **PŘESVIT LESKU (test 2)**
  - koule mezi světlem a kamerou, h = 1 → odlesk prosvítá na odvrácenou stranu
  - oprava: když `dot(N, L) < 0`, lesk = 0
- 🆕 **STÍNOVÁNÍ (jak se barva nanáší na plochy)**
  - **konstantní (flat)** = barva spočítaná v jednom bodě pro celý polygon → nejrychlejší, hranaté
  - **Gouraudovo** = barva ve **vrcholech** (vertex shader), pak se **interpoluje barva**
  - **Phongovo** = **interpoluje se normála**, osvětlení **v každém fragmentu** (fragment shader) → nejhezčí, nejpomalejší ← u nás
  - ⚠️ Phongovo **stínování** ≠ Phongův **osvětlovací model**
  - smooth vs. flat normály v modelu (Blender) = zaoblený vs. hranatý vzhled
- 🆕 **LINEÁRNÍ MAPOVÁNÍ NA INTERVAL** (barvení stromu): `x′ = a + (b − a)·(x − xmin) / (xmax − xmin)`

---

## 🧭 9) Transformace normál (P6)

- 🆕 **CO SMÍM DÁT NA NORMÁLU**
  - ✅ **rotace** – normála se má otočit s objektem
  - ❌ **translace** – normála je **směr**, posun ji nesmí změnit (proto w = 0 / `mat3`)
  - ❌ **neuniformní scale** – normála by přestala být **kolmá** na povrch
- 🆕 **TEČNÝ VEKTOR** `t = V2 − V1` se transformuje normálně: `t′ = M·t` (lineární část, `mat3`)
- 🆕 **NORMÁLOVÁ MATICE – odvození**
  - před i po transformaci musí platit kolmost: `nᵀ·t = 0` a `(N·n)ᵀ·(M·t) = 0`
  - `(A·B)ᵀ = Bᵀ·Aᵀ` → `nᵀ·Nᵀ·M·t = 0` → musí být `Nᵀ·M = I`
  - → **`N = (M⁻¹)ᵀ`** → v GLSL `transpose(inverse(mat3(modelMatrix)))`
  - pro čistou rotaci je `N = M` (protože R⁻¹ = Rᵀ)
  - jde spočítat na CPU (GLM) i GPU (shader); `transpose(inverse(mat3(V·M)))` = normála do **prostoru kamery**
- 🆕 **INVERZNÍ MATICE**: `M·M⁻¹ = I` → bod zpět do lokálního prostoru `V1 = M⁻¹·V1′` (tak jde převést i světlo a kameru do object space)
- 🆕 **TEST 4 – matice s 5 vpravo dole**
  - bod (x, y, z, 1) → (x, y, z, **5**) → po dělení w = (x/5, y/5, z/5) → objekt **5× menší** (uniformní zmenšení)
  - proto v shaderu `worldPosition = (M·v).xyz / w`
- 🆕 **PŘÍKLAD LAMBERT (P6)**: světlo L = [−2, 2, 0, **2**] → kartézsky **[−1, 1, 0]**, F = [0,0,0], n = (0,1,0)
  - l = (−1, 1, 0) → normalizovat → (−1/√2, 1/√2, 0)
  - `dot(n, l) = 1/√2 ≈ 0.7071` → bílé světlo → **Id ≈ (0.707, 0.707, 0.707)**
- 🆕 **TESTY Z PŘEDNÁŠEK** (= moje scény)
  - test 1: světlo v [0,0,0], 4 koule kolem, h = 32 → kontrola ambient, difúzní, zrcadlové složky a normál (scéna 6)
  - test 2: přesvit lesku (scéna 7)
  - test 3: více modelů a shaderů (konstantní, Lambert, Phong, Blinn), při změně okna se nic nedeformuje
  - test 4: matice s w = 5

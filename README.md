# LiteCrypt-ASCON-Algorithm-Research

報告影片播放清單:
https://youtube.com/playlist?list=PLOSnGAePofrNYvxPQvjS-4y3P-v8gXeYQ

**簡介**

Ascon 是一系列來自奧地利[格拉茨科技大學](https://www.tugraz.at/)、 [英飛凌科技](https://www.infineon.com/)、 [Lamarr Security Research](https://www.lamarr.at/)和 [Radboud 大學](https://www.ru.nl/english/)的密碼學家團隊設計用<a name="_hlk122613901"></a>於輕量級認證加密和雜湊的演算法，該演算法已在 [CAESAR competition (2014–2019)](https://competitions.cr.yp.to/caesar-submissions.html) 中勝出並被評選為主要的輕量級加密演算法，現在正在[NIST Lightweight Cryptography competition (2019–)](https://csrc.nist.gov/projects/lightweight-cryptography/finalists) 決賽競爭中。

**功能**

Ascon 提供了帶有關聯資料的認證加密（AEAD）、雜湊與拓展輸出函式。

- Authenticated encryption

使用符號 Ek,r,a,b 表示加密函數、 Dk,r,a,b 表示解密函數，其中參數 k = 金鑰長度（≤160 單位為bite）、r = 資料區塊長度（單位為bits） 、a,b = 排列輪數。

在加密函式中需要四個輸入 K, N, A ,P，其 K= key （長度為 k ，單位為bits）、N = public message number （128 bits的隨機數）、A = associated data（任意長度）、P =  plaintext（任意長度），然後加密完後會輸出 C, T ，其C = authenticated ciphertext （與P同長)、T = authentication tag（128 bits），完整函式如下表示:

Ek,r,a,bK,N,A,P=(C,T)

在解密函式中需要五個輸入 K, N, A, C, T，其K、N、A 意思與加密輸入的 K ,N ,A 相同，C、T 意思與加密輸出的 C, T 相同，加密完後輸出 P ||⊥，其 P 與加密輸入的 P 相同（當tag驗證成功時輸出）、⊥ = error （當tag驗證失敗時輸出），完整函式如下表示:

Dk,r,a,bK,N,A,C,T∈{P,⊥}

- Hashing

使用符號 Xh,r,a,b 表示雜湊函數，其中參數 h = 輸出長度限制（若 h = 0 代表無限制）、r = 資料區塊長度（單位為bits）、a, b = 排列輪數。

在雜湊函式中需要兩個輸入 M, l ，其 M =  input message （任意長度）、l = arbitrary specified length（l ≤加解密參數中的 h），然後加密完後會輸出 H，其 H = 雜湊結果（隨機長度 l ≤ h）完整函式如下表示:

Xh,r,a,bM,l=H

注意：所有的雜湊和拓展輸出均使用此函式，只是參數設置的差異，雜湊函式使用參數 h = 256、拓展輸出函式參數 h = 0。



**設置建議**

- Authenticated encryption

官方推薦的參數配置方案主要為 Ascon-128： E,D128,64,12,6，次要為 Ascon-128a：E,D128,128,12,8。

還有一個加解密配置為 Ascon-80pq ，這個配置主要是提升了金鑰長度以抵抗使用 Grover 演算法的量子攻擊，其配置為 E,D160,64,12,6。

- Hashing

官方推薦的參數配置方案主要為 Ascon-Hash：X256,64,12,12withl = 256，次要為Ascon-Hasha： X256,64,12,8 with l = 256。

- Pairing for Authenticated Encryption and Hashing

官方推薦的參數配置方案主要為Ascon-128 and Ascon-Hash（兩者要為相同的r），次要為Ascon-128a and Ascon-Hasha（兩者要為相同的 b）。

- Permutation進階設置

提供了兩個給拓展輸出使用的雜湊函式配置Ascon-Xof：X0,64,12,12、Ascon-Xofa：X0,64,12,8



**設計**

Ascon的設計目標是能夠在硬體和軟體之上都能用極少的記憶體儲存量來執行加解密，其套件內部各演算法設計原理如下：

- Ascon加解密、雜湊與拓展輸出函式

遵循於[海綿結構]()，更正確來說應該要更接近於 [SpongeWrap](https://link.springer.com/chapter/10.1007/978-3-642-28496-0_19)。

- Permutation（應用於上述功能之中）

使用迭代[替換排列網絡（SPN）]()，以在低成本情況下提供良好的密碼特性和傳播速率。

Permutation 分為3個階段：PC（Addition of Constants）、PS（Substitution Layer）、PL（Linear Diffusion Layer）

- PC：

使用一個常數值做xor運算。

- PS：

使用[Keccak（目前的SHA-3演算法）]()中的χmapping step為基礎，因χmapping是可高度平行化的，並且具有簡潔的描述和相對較少的指令。使得在軟體和硬體上都有高效能的表現。

- PL：

以[SHA-2的 Σ functions]()為基礎設計擴散功能。

**為何輕量化:**

Ascon 套件其主要在於底層的 320bits 排列僅使用按位布爾函數（and、not、xor）和 words 的旋轉。

因此排列非常適合在 64 位元平台上實現快速 bitsliced，而位元交錯允許在 32、16 和 8 位元平台上實現快速 bitsliced、Ascon 的低階 S-box 允許以較小的硬體和軟體開銷實現遮罩。

在輕量級設備進行密碼操作的場景下，Ascon 是一個很好的選擇。 也由於在軟件方面的良好表現，Ascon非常適合輕量級設備與高端伺服器通信的場景。

## Week 2 Tasks: Breaking Ciphers

### Simple Substitution Cipher
- Simple Substitution Cipher là một trong những dạng mã hóa cổ điển. Với ý tưởng ta có một bảng chữ cái gốc (A-Z), với mỗi kí tự ta thay thế nó bằng một kí tự khác trong bảng chữ cái theo một ánh xạ cố định.

##### Encrypt
- Tạo một keymap, có thể random để khó phá hơn.

```py
def generate_substitution_key():
    """Generate a random substitution key (dict)."""
    letters = list(ALPHABET)
    random.shuffle(letters)
    return dict(zip(ALPHABET, letters))

def parse_substitution_key(key_str: str):
    """
    Parse a 26-character key string into a substitution mapping.
    Example: "QWERTYUIOPASDFGHJKLZXCVBNM"
    """
    key_str = key_str.upper()
    if len(key_str) != 26 or len(set(key_str)) != 26:
        raise ValueError("Substitution key must contain 26 unique letters.")
    return dict(zip(ALPHABET, key_str))

def substitution_encrypt(text: str, key: dict) -> str:
    """Encrypt text using a substitution key (dict)."""
    return ''.join(key.get(ch, ch) for ch in text.upper())
```

##### Decrypt
```py
def substitution_decrypt(cipher: str, key: dict) -> str:
    """Decrypt text using a substitution key (dict)."""
    inv_key = {v: k for k, v in key.items()}
    return ''.join(inv_key.get(ch, ch) for ch in cipher.upper())
```

- Trong trường hợp không biết keymap, ta có thể phân tích tần suất xuất hiện của các chữ cái và map thử cái kí tự thường xuyên xuất hiện để thử tìm kiếm các từ có nghĩa.

### Caesar Cipher
- Caesar Cipher là một loại mã hóa ánh xạ 1-1 với kí tự được thay thế với một kí tự khác với độ dịch chuyển cố định `(char + shift) mod 26`.

##### Encrypt && Decrypt
```py
def caesar(text: str, shift: int = 3, decrypt: bool = False) -> str:
    """
    Caesar cipher encryption/decryption.
    Key = shift (integer).
    """
    if not isinstance(shift, int):
        raise ValueError("Shift must be an integer.")
    shift = -shift if decrypt else shift
    return ''.join(
        ALPHABET[(ALPHABET.index(ch) + shift) % M] if ch in ALPHABET else ch
        for ch in text.upper()
    )
```

- Dù không biết key shift vẫn có thể bruteforce được do độ dài bảng chữ cái chỉ có 26 nên chỉ cần thử shift 26 lần là có thể phá.

### Atbash Cipher
- Atbash Cipher là một loại mã hóa sử dụng phép thế bằng cách đảo ngược bảng chữ cái.

##### Encrypt && Decrypt
```py
def atbash(text):
    return ''.join(ALPHABET[M - 1 - ALPHABET.index(ch)] if ch in ALPHABET else ch for ch in text.upper())
```

- Vì Atbash chỉ lật ngược bảng chữ cái nên chỉ với 1 hàm atbash có thể giải mã được chính nó.

### Affine Cipher
- Affine Cipher là một dạng mã thay thế nhưng sử dụng một hàm số toán học `E(x) = (ax + b) mod 26`.

##### Encrypt && Decrypt
```py
def affine(text: str, a: int = 5, b: int = 8, decrypt: bool = False) -> str:
    """
    Affine cipher encryption/decryption.
    Key = (a, b) where gcd(a, 26) = 1.
    E(x) = (a*x + b) mod 26
    D(y) = a_inv * (y - b) mod 26
    """
    if gcd(a, M) != 1:
        raise ValueError(f"a={a} must be coprime with 26 for the Affine cipher.")

    if decrypt:
        a_inv = pow(a, -1, M)  # modular inverse of a mod 26
        return ''.join(
            ALPHABET[(a_inv * (ALPHABET.index(ch) - b)) % M] if ch in ALPHABET else ch
            for ch in text.upper()
        )
    else:
        return ''.join(
            ALPHABET[(a * ALPHABET.index(ch) + b) % M] if ch in ALPHABET else ch
            for ch in text.upper()
        )
```

- Hàm encrypt truyền vào 2 tham số a, b và ánh xạ theo công thức với a là coprime của 26.
- Biến đổi từ công thức encrypt ta được `D(y) = a_inv * (y - b) mod 26` và vì chỉ mod 26 nên có thể bruteforce để phá. (a = {1, 3, 5, 7, 9, 11, 15, 17, 19, 21, 23, 25}) -> O(12*26).

### Keyword Cipher
- Keyword Cipher cũng là một loại mã thay thế, nhưng bảng chữ cái thay thế không được chọn ngẫu nhiên hay dịch chuyển đơn giản như Caesar, mà được xây dựng dựa trên một keyword.
- Ta sẽ chọn một từ khóa: `hello`.
- Loại bỏ kí tự trùng lặp -> `helo`.
- Viết bảng chữ cái còn lại sau từ khóa, bỏ đi những kí tự đã xuất hiện.

```
key = hello -> key = helo
cipher = h e l o a b c d f g i j k m n p q r s t u v w x y z
```

##### Encrypt && Decrypt 
```py
def keyword_cipher(text: str, keyword: str = "SECRET", decrypt: bool = False, show_key: bool = False) -> str:
    """
    Keyword cipher: build substitution alphabet starting with keyword,
    then fill with remaining letters.
    Key = keyword (string, letters only).
    If show_key=True, also prints the substitution table.
    """
    # Clean keyword: uppercase, remove non-letters, drop duplicates
    keyword = ''.join(ch for ch in keyword.upper() if ch in ALPHABET)
    keyword = ''.join(sorted(set(keyword), key=keyword.index))
    
    # Build cipher alphabet
    rest = ''.join(ch for ch in ALPHABET if ch not in keyword)
    cipher_alphabet = keyword + rest

    # Build mappings
    mapping = dict(zip(ALPHABET, cipher_alphabet))
    inv_mapping = {v: k for k, v in mapping.items()}

    # Optionally display key matrix
    if show_key:
        print("Plain : " + " ".join(ALPHABET))
        print("Cipher: " + " ".join(cipher_alphabet))

    # Encrypt / Decrypt
    if decrypt:
        return ''.join(inv_mapping.get(ch, ch) for ch in text.upper())
    else:
        return ''.join(mapping.get(ch, ch) for ch in text.upper())
```

- Để decrypt thì chỉ cần mapping ngược lại.

### Homophonic Substitution
- Homophonic Substitution là loại mã ánh xạ nhưng đặc biệt hơn với một chữ cái có thể được mã hóa thành nhiều ký hiệu khác nhau.

##### Encrypt 
```py
def load_homophonic_mapping(file_path: str = None):
    """
    Load homophonic mapping from a JSON file.
    If no file is given, use a default built-in mapping.
    """
    if file_path:
        with open(file_path, "r", encoding="utf-8") as f:
            mapping = json.load(f)
    else:
        # Default demo mapping (same as before)
        mapping = {
            'E': ['1', '!', 'Q', '9', 'X', '3'],
            'T': ['2', '$', 'Y', '+'],
            'A': ['@', '4', '7'],
            'O': ['0', 'O', '*'],
            'I': ['|', 'i', '8'],
            'N': ['~', 'n', '^'],
            'S': ['$', '5', '%'],
            'H': ['#', 'h', '&'],
            'R': ['R', '?', '4'],
            'D': ['D', ')'],
            'L': ['L', '/'],
            'C': ['C', '(', '['],
            'U': ['U', '_'],
            'M': ['M', '='],
            'W': ['W', '{'],
            'F': ['F', '}'],
            'G': ['G', ':'],
            'Y': ['Y', ';'],
            'P': ['P', '.'],
            'B': ['B', ','],
            'V': ['V', '<'],
            'K': ['K', '>'],
            'J': ['J', '`'],
            'X': ['X', '"'],
            'Q': ['Q', "'"],
            'Z': ['Z', '\\']
        }

    # Ensure all 26 letters exist (fallback self-mapping)
    for ch in ALPHABET:
        if ch not in mapping:
            mapping[ch] = [ch]
    return mapping

def homophonic_encrypt(text: str, mapping: dict) -> str:
    """Encrypt using homophonic substitution."""
    result = []
    for ch in text.upper():
        if ch in mapping:
            result.append(random.choice(mapping[ch]))
        else:
            result.append(ch)  # keep non-letters as is
    return ''.join(result)
```

##### Decrypt 
```py
def homophonic_decrypt(cipher: str, mapping: dict) -> str:
    """Decrypt homophonic ciphertext (many-to-one mapping)."""
    reverse = {}
    for k, vals in mapping.items():
        for v in vals:
            reverse[v] = k
    return ''.join(reverse.get(ch, ch) for ch in cipher)
```
- Nếu không có bảng mapping từ đầu thì việc đếm tần suất và dò từ sẽ khá khó khăn và mất thời gian.

### Pigpen Cipher
- Pigpen cipher là một loại monoalphabetic substitution cipher ánh xạ kí tự thành một kí hiệu hình học.
<img width="230" height="230" alt="image" src="https://github.com/user-attachments/assets/86df3691-f0a0-4001-9898-ca1c73853105" />

##### Encrypt 
```py

# Default simple placeholder Pigpen map (A=∆, B=⊡, ... etc.)
DEFAULT_PIGPEN_SYMBOLS = [
    "∆", "⊡", "□", "⊞", "⊟", "⊠", "◈", "◇", "◆", "◉", 
    "○", "◍", "◎", "●", "◐", "◑", "◒", "◓", "◔", "◕",
    "★", "✦", "✧", "✩", "✪", "✫"
]
DEFAULT_PIGPEN_MAP = dict(zip(ALPHABET, DEFAULT_PIGPEN_SYMBOLS))


def build_pigpen_map(custom_symbols: str = None):
    """
    Build Pigpen mapping. If custom_symbols is provided,
    it must contain 26 unique characters.
    """
    if custom_symbols:
        if len(custom_symbols) != 26:
            raise ValueError("Pigpen key must have exactly 26 symbols/characters.")
        return dict(zip(ALPHABET, list(custom_symbols)))
    else:
        return DEFAULT_PIGPEN_MAP


def pigpen_encrypt(text: str, mapping: dict) -> str:
    """Encrypt using Pigpen cipher with mapping."""
    return ''.join(mapping.get(ch, ch) for ch in text.upper())
```

##### Decrypt
```py
def pigpen_decrypt(cipher: str, mapping: dict) -> str:
    """Decrypt Pigpen ciphertext (symbol-to-letter)."""
    inv = {v: k for k, v in mapping.items()}
    return ''.join(inv.get(ch, ch) for ch in cipher)
```

### ROT-N Variants
- Rot-N là một loại mã hóa thay thế giống với Caesar nhưng với N là một shift cố định.

##### Encrypt && Decrypt

```py
def rotN(text, n=13):
    return ''.join(ALPHABET[(ALPHABET.index(ch) + n) % M] if ch in ALPHABET else ch for ch in text.upper())
```

### A1Z26 Cipher 
- A1Z26 là một dạng Numeric Cipher biểu diễn chữ cái bằng số bằng cách ánh xạ A=1, B=2, …, Z=26.

##### Encrypt && Decrypt
```py
def a1z26(text, decrypt=False):
    if decrypt:
        return ''.join(ALPHABET[int(p)-1] if p.isdigit() else p for p in text.split())
    else:
        return ' '.join(str(ALPHABET.index(ch) + 1) if ch in ALPHABET else ch for ch in text.upper())
```

### Rail Fence Cipher 
- Rail Fence Cipher là một dạng transposition cipher tức là nó không thay đổi ký tự gốc mà chỉ xáo trộn vị trí của chúng dựa trên một quy luật nhất định.
- Cách hoạt động:
```
plaintext: im harry potter
```
- Ta viết chữ theo kiểu zig-zag xuống 3 hàng:
```
i . . . r . . . o . . . r 
. m . a . r . p . t . e . 
. . h . . . y . . . t . . 
```
- Sau đó ta đọc theo từng hàng:
```
ciphertext: irormarptehyt 
```

##### Encrypt 
```py
def rail_fence_encrypt(text: str, rails: int) -> str:
    text = text.replace(" ", "").upper()
    fence = [[] for _ in range(rails)]
    rail = 0
    var = 1
    for ch in text:
        fence[rail].append(ch)
        rail += var
        if rail == 0 or rail == rails - 1:
            var = -var
    return ''.join(''.join(row) for row in fence)
```
- rails chính là số hàng.

##### Decrypt
```py
def rail_fence_decrypt(cipher: str, rails: int) -> str:
    cipher = cipher.upper()
    # Build zigzag pattern indices
    pattern = list(range(rails)) + list(range(rails-2, 0, -1))
    pat_len = len(pattern)
    n = len(cipher)
    indices = [pattern[i % pat_len] for i in range(n)]
    # Count letters per rail
    rail_counts = [indices.count(r) for r in range(rails)]
    # Split cipher into rails
    pos = 0
    rail_strs = []
    for count in rail_counts:
        rail_strs.append(list(cipher[pos:pos+count]))
        pos += count
    # Reconstruct
    result = []
    rail_positions = [0]*rails
    for r in indices:
        result.append(rail_strs[r][rail_positions[r]])
        rail_positions[r] += 1
    return ''.join(result)
```
- Để phá mà không biết rails thì chỉ cần bruteforce từng rail một cho tới khi có nghĩa.

### Columnar Transposition
- Columnar Transposition Cipher cũng thuộc kiểu transposition cipher, nó dựa trên một bảng và một khóa là thứ tự sắp xếp các cột.
- Cách hoạt động:
```
key = login
      35124
```
- Chọn một khoá và gán số thứ tự theo thứ tự bảng chữ cái.

```
plaintext: im not harry potter
```
- Với khóa `login` có 5 kí tự ta viết 5 cột.

```
i m n o t
h a r r y
p o t t e
r x x x x
```

- Nếu thiểu thì padding thêm vào, ở đây là padding `x`.
- Đọc từng cột theo thứ tự ưu tiên của khóa.
```
plaintext: nrtx ortx ihpr tyex maox 
```

##### Encrypt
```py
def columnar_encrypt(text: str, key: str) -> str:
    text = text.replace(" ", "").upper()
    key = key.upper()
    ncols = len(key)
    nrows = math.ceil(len(text) / ncols)
    # pad with X
    padded = text.ljust(nrows*ncols, 'X')
    # build matrix
    matrix = [list(padded[i*ncols:(i+1)*ncols]) for i in range(nrows)]
    # order columns by key
    order = sorted(range(len(key)), key=lambda i: key[i])
    out = []
    for c in order:
        for r in range(nrows):
            out.append(matrix[r][c])
    return ''.join(out)
```

##### Decrypt 
```py
def columnar_decrypt(cipher: str, key: str) -> str:
    key = key.upper()
    ncols = len(key)
    nrows = math.ceil(len(cipher) / ncols)
    order = sorted(range(len(key)), key=lambda i: key[i])
    # allocate matrix
    matrix = [[None]*ncols for _ in range(nrows)]
    pos = 0
    for c in order:
        for r in range(nrows):
            if pos < len(cipher):
                matrix[r][c] = cipher[pos]
                pos += 1
    # read row-wise
    out = ''.join(matrix[r][c] for r in range(nrows) for c in range(ncols))
    return out.rstrip('X')
```

### Scytale Cipher
- Scytale Cipher có cách encrypt khá giống Columnar Tranposition, tạo bảng tương tự. Nhưng sẽ đọc từng cột một từ trái sang phải.
```
plaintext = im not harry potter
i m n o t
h a r r y
p o t t e
r x x x x
-> ciphertext = ihpr maox nrtx ortx tyex
```

##### Encrypt
```py
def scytale_encrypt(text: str, diameter: int) -> str:
    text = text.replace(" ", "").upper()
    nrows = math.ceil(len(text) / diameter)
    padded = text.ljust(nrows*diameter, 'X')
    out = []
    for c in range(diameter):
        for r in range(nrows):
            out.append(padded[r*diameter + c])
    return ''.join(out)
```

##### Decrypt 
```py
def scytale_decrypt(cipher: str, diameter: int) -> str:
    nrows = math.ceil(len(cipher) / diameter)
    out = []
    pos = 0
    grid = [[None]*diameter for _ in range(nrows)]
    for c in range(diameter):
        for r in range(nrows):
            if pos < len(cipher):
                grid[r][c] = cipher[pos]
                pos += 1
    for r in range(nrows):
        for c in range(diameter):
            if grid[r][c]:
                out.append(grid[r][c])
    return ''.join(out).rstrip('X')
```

### Vigenère Cipher
- Vigenère Cipher là một dạng polyalphabetic substitution cipher. Nó được mã hóa bằng hàm `F(x) = (idx(x) + shift) mod 26`. Với `idx(x)` là chỉ số của kí tự `x` trong bảng chữ cái và `shift` là giá trị `idx(key)` biến đổi.

##### Encrypt && Decrypt
```py
def vigenere(text: str, key: str, decrypt: bool = False) -> str:
    key = ''.join(ch for ch in key.upper() if ch in ALPHABET)
    if not key:
        raise ValueError("Key must contain alphabetic characters.")
    key_indices = [ALPHABET.index(k) for k in key]
    result = []
    j = 0
    for ch in text.upper():
        if ch in ALPHABET:
            shift = key_indices[j % len(key_indices)]
            if decrypt:
                shift = -shift
            idx = (ALPHABET.index(ch) + shift) % M
            result.append(ALPHABET[idx])
            j += 1
        else:
            result.append(ch)
    return ''.join(result)
```
- 



### Beaufort Cipher
### Autokey Cipher
### Playfair Cipher
### Hill Cipher

### Breaking Text
```
◔◆●□⊟ ◕◇⊟ ◓⊟◍⊟∆◔⊟ ◐⊠ ◕◇⊟ ⊠◆◓◔◕ ●◐✦⊟◍, ◇∆◓◓✪ ◑◐◕◕⊟◓ ∆●⊞ ◕◇⊟ ◑◇◆◍◐◔◐◑◇⊟◓'◔ ◔◕◐●⊟, ◐● 26 ◉★●⊟ 1997, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ⊠◐★●⊞ ◆◎◎⊟●◔⊟ ◑◐◑★◍∆◓◆◕✪ ∆●⊞ □◐◎◎⊟◓□◆∆◍ ◔★□□⊟◔◔ ✧◐◓◍⊞✧◆⊞⊟. ◕◇⊟✪ ◇∆✦⊟ ∆◕◕◓∆□◕⊟⊞ ∆ ✧◆⊞⊟ ∆⊞★◍◕ ∆★⊞◆⊟●□⊟ ∆◔ ✧⊟◍◍ ∆◔ ✪◐★●◈⊟◓ ◓⊟∆⊞⊟◓◔ ∆●⊞ ∆◓⊟ ✧◆⊞⊟◍✪ □◐●◔◆⊞⊟◓⊟⊞ □◐◓●⊟◓◔◕◐●⊟◔ ◐⊠ ◎◐⊞⊟◓● ◍◆◕⊟◓∆◕★◓⊟,[3][4] ◕◇◐★◈◇ ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◓⊟□⊟◆✦⊟⊞ ◎◆✩⊟⊞ ◓⊟✦◆⊟✧◔ ⊠◓◐◎ □◓◆◕◆□◔ ∆●⊞ ◍◆◕⊟◓∆◓✪ ◔□◇◐◍∆◓◔. ∆◔ ◐⊠ ⊠⊟⊡◓★∆◓✪ 2023, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◔◐◍⊞ ◎◐◓⊟ ◕◇∆● 600 ◎◆◍◍◆◐● □◐◑◆⊟◔ ✧◐◓◍⊞✧◆⊞⊟, ◎∆○◆●◈ ◕◇⊟◎ ◕◇⊟ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ◆● ◇◆◔◕◐◓✪, ∆✦∆◆◍∆⊡◍⊟ ◆● ⊞◐✫⊟●◔ ◐⊠ ◍∆●◈★∆◈⊟◔. ◕◇⊟ ◍∆◔◕ ⊠◐★◓ ⊡◐◐○◔ ∆◍◍ ◔⊟◕ ◓⊟□◐◓⊞◔ ∆◔ ◕◇⊟ ⊠∆◔◕⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○◔ ◆● ◇◆◔◕◐◓✪, ✧◆◕◇ ◕◇⊟ ⊠◆●∆◍ ◆●◔◕∆◍◎⊟●◕ ◔⊟◍◍◆●◈ ◓◐★◈◇◍✪ 2.7 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ○◆●◈⊞◐◎ ∆●⊞ 8.3 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ◔◕∆◕⊟◔ ✧◆◕◇◆● ◕✧⊟●◕✪-⊠◐★◓ ◇◐★◓◔ ◐⊠ ◆◕◔ ◓⊟◍⊟∆◔⊟. ◆◕ ◇◐◍⊞◔ ◕◇⊟ ◈★◆●●⊟◔◔ ✧◐◓◍⊞ ◓⊟□◐◓⊞ ⊠◐◓ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ⊠◐◓ □◇◆◍⊞◓⊟●.
```

- Ta có thể thấy đây là monoalphabetic substitution cipher.
- Sử dụng chuỗi tần suất các kí tự giảm dần `ETAOINSHRDLUCMWFGYPBVKJXQZ` và phân tích text có được.
```py
from collections import Counter

ciphertext = """◔◆●□⊟ ◕◇⊟ ◓⊟◍⊟∆◔⊟ ◐⊠ ◕◇⊟ ⊠◆◓◔◕ ●◐✦⊟◍, ◇∆◓◓✪ ◑◐◕◕⊟◓ ∆●⊞ ◕◇⊟ ◑◇◆◍◐◔◐◑◇⊟◓'◔ ◔◕◐●⊟, ◐● 26 ◉★●⊟ 1997, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ⊠◐★●⊞ ◆◎◎⊟●◔⊟ ◑◐◑★◍∆◓◆◕✪ ∆●⊞ □◐◎◎⊟◓□◆∆◍ ◔★□□⊟◔◔ ✧◐◓◍⊞✧◆⊞⊟. ◕◇⊟✪ ◇∆✦⊟ ∆◕◕◓∆□◕⊟⊞ ∆ ✧◆⊞⊟ ∆⊞★◍◕ ∆★⊞◆⊟●□⊟ ∆◔ ✧⊟◍◍ ∆◔ ✪◐★●◈⊟◓ ◓⊟∆⊞⊟◓◔ ∆●⊞ ∆◓⊟ ✧◆⊞⊟◍✪ □◐●◔◆⊞⊟◓⊟⊞ □◐◓●⊟◓◔◕◐●⊟◔ ◐⊠ ◎◐⊞⊟◓● ◍◆◕⊟◓∆◕★◓⊟,[3][4] ◕◇◐★◈◇ ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◓⊟□⊟◆✦⊟⊞ ◎◆✩⊟⊞ ◓⊟✦◆⊟✧◔ ⊠◓◐◎ □◓◆◕◆□◔ ∆●⊞ ◍◆◕⊟◓∆◓✪ ◔□◇◐◍∆◓◔. ∆◔ ◐⊠ ⊠⊟⊡◓★∆◓✪ 2023, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◔◐◍⊞ ◎◐◓⊟ ◕◇∆● 600 ◎◆◍◍◆◐● □◐◑◆⊟◔ ✧◐◓◍⊞✧◆⊞⊟, ◎∆○◆●◈ ◕◇⊟◎ ◕◇⊟ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ◆● ◇◆◔◕◐◓✪, ∆✦∆◆◍∆⊡◍⊟ ◆● ⊞◐✫⊟●◔ ◐⊠ ◍∆●◈★∆◈⊟◔. ◕◇⊟ ◍∆◔◕ ⊠◐★◓ ⊡◐◐○◔ ∆◍◍ ◔⊟◕ ◓⊟□◐◓⊞◔ ∆◔ ◕◇⊟ ⊠∆◔◕⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○◔ ◆● ◇◆◔◕◐◓✪, ✧◆◕◇ ◕◇⊟ ⊠◆●∆◍ ◆●◔◕∆◍◎⊟●◕ ◔⊟◍◍◆●◈ ◓◐★◈◇◍✪ 2.7 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ○◆●◈⊞◐◎ ∆●⊞ 8.3 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ◔◕∆◕⊟◔ ✧◆◕◇◆● ◕✧⊟●◕✪-⊠◐★◓ ◇◐★◓◔ ◐⊠ ◆◕◔ ◓⊟◍⊟∆◔⊟. ◆◕ ◇◐◍⊞◔ ◕◇⊟ ◈★◆●●⊟◔◔ ✧◐◓◍⊞ ◓⊟□◐◓⊞ ⊠◐◓ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ⊠◐◓ □◇◆◍⊞◓⊟●."""

cipher_symbols = "◔◆●□⊟◕◇◓◍∆◐⊠✧◑◎★◉⊞◈✪⊡○✦✩✫"  

english_freq = "ETAOINSHRDLUCMWFGYPBVKJXQZ"

mapping = {cipher_symbols[i]: english_freq[i] for i in range(len(cipher_symbols))}

counter = Counter(ciphertext)
for ch in [' ', '\n']:
    counter.pop(ch, None)

print("Frequency")
for sym, freq in counter.most_common():
    print(f"{sym} : {freq}")

print("\nMapping")
for sym, eng in mapping.items():
    print(f"{sym} -> {eng}")

decoded = "".join(mapping.get(ch, ch) for ch in ciphertext)
print("\nDecoded\n")
print(decoded)
```

```
ETAOI NSI HIRIDEI LU NSI UTHEN ALJIR, SDHHB MLNNIH DAY NSI MSTRLELMSIH'E ENLAI, LA 26 GFAI 1997, NSI VLLKE SDJI ULFAY TWWIAEI MLMFRDHTNB DAY OLWWIHOTDR EFOOIEE CLHRYCTYI. NSIB SDJI DNNHDONIY D CTYI DYFRN DFYTIAOI DE CIRR DE BLFAPIH HIDYIHE DAY DHI CTYIRB OLAETYIHIY OLHAIHENLAIE LU WLYIHA RTNIHDNFHI,[3][4] NSLFPS NSI VLLKE SDJI HIOITJIY WTXIY HIJTICE UHLW OHTNTOE DAY RTNIHDHB EOSLRDHE. DE LU UIVHFDHB 2023, NSI VLLKE SDJI ELRY WLHI NSDA 600 WTRRTLA OLMTIE CLHRYCTYI, WDKTAP NSIW NSI VIEN-EIRRTAP VLLK EIHTIE TA STENLHB, DJDTRDVRI TA YLQIAE LU RDAPFDPIE. NSI RDEN ULFH VLLKE DRR EIN HIOLHYE DE NSI UDENIEN-EIRRTAP VLLKE TA STENLHB, CTNS NSI UTADR TAENDRWIAN EIRRTAP HLFPSRB 2.7 WTRRTLA OLMTIE TA NSI FATNIY KTAPYLW DAY 8.3 WTRRTLA OLMTIE TA NSI FATNIY ENDNIE CTNSTA NCIANB-ULFH SLFHE LU TNE HIRIDEI. TN SLRYE NSI PFTAAIEE CLHRY HIOLHY ULH VIEN-EIRRTAP VLLK EIHTIE ULH OSTRYHIA.
```

- Nhìn qua ta có thể thấy từ `NSI` xuất hiện khá thường xuyên, ta có thể đoán `NSI` -> `THE`, map thử ta được.

```
INAOE THE SEREDIE LU THE UNSIT ALJER, HDSSB MLTTES DAY THE MHNRLILMHES'I ITLAE, LA 26 GFAE 1997, THE VLLKI HDJE ULFAY NWWEAIE MLMFRDSNTB DAY OLWWESONDR IFOOEII CLSRYCNYE. THEB HDJE DTTSDOTEY D CNYE DYFRT DFYNEAOE DI CERR DI BLFAPES SEDYESI DAY DSE CNYERB OLAINYESEY OLSAESITLAEI LU WLYESA RNTESDTFSE,[3][4] THLFPH THE VLLKI HDJE SEOENJEY WNXEY SEJNECI USLW OSNTNOI DAY RNTESDSB IOHLRDSI. DI LU UEVSFDSB 2023, THE VLLKI HDJE ILRY WLSE THDA 600 WNRRNLA OLMNEI CLSRYCNYE, WDKNAP THEW THE VEIT-IERRNAP VLLK IESNEI NA HNITLSB, DJDNRDVRE NA YLQEAI LU RDAPFDPEI. THE RDIT ULFS VLLKI DRR IET SEOLSYI DI THE UDITEIT-IERRNAP VLLKI NA HNITLSB, CNTH THE UNADR NAITDRWEAT IERRNAP SLFPHRB 2.7 WNRRNLA OLMNEI NA THE FANTEY KNAPYLW DAY 8.3 WNRRNLA OLMNEI NA THE FANTEY ITDTEI CNTHNA TCEATB-ULFS HLFSI LU NTI SEREDIE. NT HLRYI THE PFNAAEII CLSRY SEOLSY ULS VEIT-IERRNAP VLLK IESNEI ULS OHNRYSEA.
```

- `LA 26 GFAE 1997` ở đây ta có thể đoán `LA` là `ON` và `GFAE` có thể là một tháng nào đó có 4 kí tự `JUNE` hoặc `JULY` nhưng với chữ `E` có thể đã đúng vì chữ `THE` nên ở đây tháng có thể là `JUNE`.
- `THEB HDJE DTTSDOTEY D CNYE` kí tự `D` có thể sẽ là mạo từ `A`.

```
IDNLE THE SEREAIE OF THE FDSIT NOGER, HASSB MOTTES ANY THE MHDROIOMHES'I ITONE, ON 26 JUNE 1997, THE VOOKI HAGE FOUNY DWWENIE MOMURASDTB ANY LOWWESLDAR IULLEII COSRYCDYE. THEB HAGE ATTSALTEY A CDYE AYURT AUYDENLE AI CERR AI BOUNPES SEAYESI ANY ASE CDYERB LONIDYESEY LOSNESITONEI OF WOYESN RDTESATUSE,[3][4] THOUPH THE VOOKI HAGE SELEDGEY WDXEY SEGDECI FSOW LSDTDLI ANY RDTESASB ILHORASI. AI OF FEVSUASB 2023, THE VOOKI HAGE IORY WOSE THAN 600 WDRRDON LOMDEI COSRYCDYE, WAKDNP THEW THE VEIT-IERRDNP VOOK IESDEI DN HDITOSB, AGADRAVRE DN YOQENI OF RANPUAPEI. THE RAIT FOUS VOOKI ARR IET SELOSYI AI THE FAITEIT-IERRDNP VOOKI DN HDITOSB, CDTH THE FDNAR DNITARWENT IERRDNP SOUPHRB 2.7 WDRRDON LOMDEI DN THE UNDTEY KDNPYOW ANY 8.3 WDRRDON LOMDEI DN THE UNDTEY ITATEI CDTHDN TCENTB-FOUS HOUSI OF DTI SEREAIE. DT HORYI THE PUDNNEII COSRY SELOSY FOS VEIT-IERRDNP VOOK IESDEI FOS LHDRYSEN.
```

- `THE VOOKI HAGE FOUNY` có thể thấy `HAGE` có thể là `HAVE` và `FOUNY` có thể là `FOUND`

```
IYNLE THE SEREAIE OF THE FYSIT NOVER, HASSB MOTTES AND THE MHYROIOMHES'I ITONE, ON 26 JUNE 1997, THE GOOKI HAVE FOUND YWWENIE MOMURASYTB AND LOWWESLYAR IULLEII COSRDCYDE. THEB HAVE ATTSALTED A CYDE ADURT AUDYENLE AI CERR AI BOUNPES SEADESI AND ASE CYDERB LONIYDESED LOSNESITONEI OF WODESN RYTESATUSE,[3][4] THOUPH THE GOOKI HAVE SELEYVED WYXED SEVYECI FSOW LSYTYLI AND RYTESASB ILHORASI. AI OF FEGSUASB 2023, THE GOOKI HAVE IORD WOSE THAN 600 WYRRYON LOMYEI COSRDCYDE, WAKYNP THEW THE GEIT-IERRYNP GOOK IESYEI YN HYITOSB, AVAYRAGRE YN DOQENI OF RANPUAPEI. THE RAIT FOUS GOOKI ARR IET SELOSDI AI THE FAITEIT-IERRYNP GOOKI YN HYITOSB, CYTH THE FYNAR YNITARWENT IERRYNP SOUPHRB 2.7 WYRRYON LOMYEI YN THE UNYTED KYNPDOW AND 8.3 WYRRYON LOMYEI YN THE UNYTED ITATEI CYTHYN TCENTB-FOUS HOUSI OF YTI SEREAIE. YT HORDI THE PUYNNEII COSRD SELOSD FOS GEIT-IERRYNP GOOK IESYEI FOS LHYRDSEN.
```

- `UNYTED KYNPDOW` sẽ là `UNITED KINGDOM` và `LOMYEI YN THE` có thể là `IN THE`.

```
YINLE THE SEREAYE OF THE FISYT NOVER, HASSB WOTTES AND THE WHIROYOWHES'Y YTONE, ON 26 JUNE 1997, THE POOKY HAVE FOUND IMMENYE WOWURASITB AND LOMMESLIAR YULLEYY COSRDCIDE. THEB HAVE ATTSALTED A CIDE ADURT AUDIENLE AY CERR AY BOUNGES SEADESY AND ASE CIDERB LONYIDESED LOSNESYTONEY OF MODESN RITESATUSE,[3][4] THOUGH THE POOKY HAVE SELEIVED MIXED SEVIECY FSOM LSITILY AND RITESASB YLHORASY. AY OF FEPSUASB 2023, THE POOKY HAVE YORD MOSE THAN 600 MIRRION LOWIEY COSRDCIDE, MAKING THEM THE PEYT-YERRING POOK YESIEY IN HIYTOSB, AVAIRAPRE IN DOQENY OF RANGUAGEY. THE RAYT FOUS POOKY ARR YET SELOSDY AY THE FAYTEYT-YERRING POOKY IN HIYTOSB, CITH THE FINAR INYTARMENT YERRING SOUGHRB 2.7 MIRRION LOWIEY IN THE UNITED KINGDOM AND 8.3 MIRRION LOWIEY IN THE UNITED YTATEY CITHIN TCENTB-FOUS HOUSY OF ITY SEREAYE. IT HORDY THE GUINNEYY COSRD SELOSD FOS PEYT-YERRING POOK YESIEY FOS LHIRDSEN.
```

- `HAVE SELEIVED MIXED SEVIECY FSOM` có thể là `HAVE RELEIVED MIXED SEVIECY FROM`, `MOSE THAN 600` thành `MORE`.
- `FISYT NOVER` có thể đoán là `FIRST NOVEL` và vì có thể đang nhắc tới tiểu thuyết, `HASSB WOTTES` nhìn khá giống `HARRY POTTER`.

```
SINBE THE RELEASE OF THE FIRST NOVEL, HARRY POTTER AND THE PHILOSOPHER'S STONE, ON 26 JUNE 1997, THE WOOKS HAVE FOUND IMMENSE POPULARITY AND BOMMERBIAL SUBBESS CORLDCIDE. THEY HAVE ATTRABTED A CIDE ADULT AUDIENBE AS CELL AS YOUNGER READERS AND ARE CIDELY BONSIDERED BORNERSTONES OF MODERN LITERATURE,[3][4] THOUGH THE WOOKS HAVE REBEIVED MIXED REVIECS FROM BRITIBS AND LITERARY SBHOLARS. AS OF FEWRUARY 2023, THE WOOKS HAVE SOLD MORE THAN 600 MILLION BOPIES CORLDCIDE, MAKING THEM THE WEST-SELLING WOOK SERIES IN HISTORY, AVAILAWLE IN DOQENS OF LANGUAGES. THE LAST FOUR WOOKS ALL SET REBORDS AS THE FASTEST-SELLING WOOKS IN HISTORY, CITH THE FINAL INSTALMENT SELLING ROUGHLY 2.7 MILLION BOPIES IN THE UNITED KINGDOM AND 8.3 MILLION BOPIES IN THE UNITED STATES CITHIN TCENTY-FOUR HOURS OF ITS RELEASE. IT HOLDS THE GUINNESS CORLD REBORD FOR WEST-SELLING WOOK SERIES FOR BHILDREN.
```

- Tới đây ta có thể đọc được kha khá và có thể hoàn thành mapping.

```py
from collections import Counter

ciphertext = """◔◆●□⊟ ◕◇⊟ ◓⊟◍⊟∆◔⊟ ◐⊠ ◕◇⊟ ⊠◆◓◔◕ ●◐✦⊟◍, ◇∆◓◓✪ ◑◐◕◕⊟◓ ∆●⊞ ◕◇⊟ ◑◇◆◍◐◔◐◑◇⊟◓'◔ ◔◕◐●⊟, ◐● 26 ◉★●⊟ 1997, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ⊠◐★●⊞ ◆◎◎⊟●◔⊟ ◑◐◑★◍∆◓◆◕✪ ∆●⊞ □◐◎◎⊟◓□◆∆◍ ◔★□□⊟◔◔ ✧◐◓◍⊞✧◆⊞⊟. ◕◇⊟✪ ◇∆✦⊟ ∆◕◕◓∆□◕⊟⊞ ∆ ✧◆⊞⊟ ∆⊞★◍◕ ∆★⊞◆⊟●□⊟ ∆◔ ✧⊟◍◍ ∆◔ ✪◐★●◈⊟◓ ◓⊟∆⊞⊟◓◔ ∆●⊞ ∆◓⊟ ✧◆⊞⊟◍✪ □◐●◔◆⊞⊟◓⊟⊞ □◐◓●⊟◓◔◕◐●⊟◔ ◐⊠ ◎◐⊞⊟◓● ◍◆◕⊟◓∆◕★◓⊟,[3][4] ◕◇◐★◈◇ ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◓⊟□⊟◆✦⊟⊞ ◎◆✩⊟⊞ ◓⊟✦◆⊟✧◔ ⊠◓◐◎ □◓◆◕◆□◔ ∆●⊞ ◍◆◕⊟◓∆◓✪ ◔□◇◐◍∆◓◔. ∆◔ ◐⊠ ⊠⊟⊡◓★∆◓✪ 2023, ◕◇⊟ ⊡◐◐○◔ ◇∆✦⊟ ◔◐◍⊞ ◎◐◓⊟ ◕◇∆● 600 ◎◆◍◍◆◐● □◐◑◆⊟◔ ✧◐◓◍⊞✧◆⊞⊟, ◎∆○◆●◈ ◕◇⊟◎ ◕◇⊟ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ◆● ◇◆◔◕◐◓✪, ∆✦∆◆◍∆⊡◍⊟ ◆● ⊞◐✫⊟●◔ ◐⊠ ◍∆●◈★∆◈⊟◔. ◕◇⊟ ◍∆◔◕ ⊠◐★◓ ⊡◐◐○◔ ∆◍◍ ◔⊟◕ ◓⊟□◐◓⊞◔ ∆◔ ◕◇⊟ ⊠∆◔◕⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○◔ ◆● ◇◆◔◕◐◓✪, ✧◆◕◇ ◕◇⊟ ⊠◆●∆◍ ◆●◔◕∆◍◎⊟●◕ ◔⊟◍◍◆●◈ ◓◐★◈◇◍✪ 2.7 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ○◆●◈⊞◐◎ ∆●⊞ 8.3 ◎◆◍◍◆◐● □◐◑◆⊟◔ ◆● ◕◇⊟ ★●◆◕⊟⊞ ◔◕∆◕⊟◔ ✧◆◕◇◆● ◕✧⊟●◕✪-⊠◐★◓ ◇◐★◓◔ ◐⊠ ◆◕◔ ◓⊟◍⊟∆◔⊟. ◆◕ ◇◐◍⊞◔ ◕◇⊟ ◈★◆●●⊟◔◔ ✧◐◓◍⊞ ◓⊟□◐◓⊞ ⊠◐◓ ⊡⊟◔◕-◔⊟◍◍◆●◈ ⊡◐◐○ ◔⊟◓◆⊟◔ ⊠◐◓ □◇◆◍⊞◓⊟●."""

cipher_symbols = "◔◆●□⊟◕◇◓◍∆◐⊠✧◑◎★◉⊞◈✪⊡○✦✩✫"  

english_freq = "SINCETHRLAOFWPMUJDGYBKVXZQ"

mapping = {cipher_symbols[i]: english_freq[i] for i in range(len(cipher_symbols))}

counter = Counter(ciphertext)
for ch in [' ', '\n']:
    counter.pop(ch, None)

print("Frequency")
for sym, freq in counter.most_common():
    print(f"{sym} : {freq}")

print("\nMapping")
for sym, eng in mapping.items():
    print(f"{sym} -> {eng}")

decoded = "".join(mapping.get(ch, ch) for ch in ciphertext)
print("\nDecoded\n")
print(decoded)
```

```
SINCE THE RELEASE OF THE FIRST NOVEL, HARRY POTTER AND THE PHILOSOPHER'S STONE, ON 26 JUNE 1997, THE BOOKS HAVE FOUND IMMENSE POPULARITY AND COMMERCIAL SUCCESS WORLDWIDE. THEY HAVE ATTRACTED A WIDE ADULT AUDIENCE AS WELL AS YOUNGER READERS AND ARE WIDELY CONSIDERED CORNERSTONES OF MODERN LITERATURE,[3][4] THOUGH THE BOOKS HAVE RECEIVED MIXED REVIEWS FROM CRITICS AND LITERARY SCHOLARS. AS OF FEBRUARY 2023, THE BOOKS HAVE SOLD MORE THAN 600 MILLION COPIES WORLDWIDE, MAKING THEM THE BEST-SELLING BOOK SERIES IN HISTORY, AVAILABLE IN DOZENS OF LANGUAGES. THE LAST FOUR BOOKS ALL SET RECORDS AS THE FASTEST-SELLING BOOKS IN HISTORY, WITH THE FINAL INSTALMENT SELLING ROUGHLY 2.7 MILLION COPIES IN THE UNITED KINGDOM AND 8.3 MILLION COPIES IN THE UNITED STATES WITHIN TWENTY-FOUR HOURS OF ITS RELEASE. IT HOLDS THE GUINNESS WORLD RECORD FOR BEST-SELLING BOOK SERIES FOR CHILDREN.
```






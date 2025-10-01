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
### Columnar Transposition
### Scytale Cipher
### Vigenère Cipher
### Beaufort Cipher
### Autokey Cipher
### Playfair Cipher
### Hill Cipher

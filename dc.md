# DC LAB — ALL C EXPERIMENTS (LABS 5–8)

**ONE MARKDOWN FILE | 11 COMPLETE C PROGRAMS | INPUT + EXPECTED OUTPUT FOR EVERY EXPERIMENT**

> GitHub-ready codebook. Every experiment starts with a large `====` heading. No comments interrupt the C programs.

**How to use:** Open this `.md` file on GitHub, choose one experiment, copy the entire `c` code block into a new `.c` file, then compile it. Each code block has its own `main()` and must run separately. Below each program, enter the values in the **INPUT** section, pressing Enter after each value, and compare with the **EXPECTED OUTPUT**. The output blocks show what appears in an interactive terminal, including the values you type.

```bash
gcc -std=c11 -Wall -Wextra your_program.c -o program
./program
```

**Study priority:** EASY → MEDIUM → HARD. Labs 1–4 use Packet Tracer, so this file contains the C experiments from Labs 5–8 only.

**Lab 6 warning:** The PDF’s standard 4B/5B lookup table and the custom even-parity 4B/5B mapping are *different algorithms*. The table is not guaranteed to detect all single-bit errors; parity mapping does not satisfy every run-length restriction.

---

```text
=========================================================================
LAB 05  |  EXPERIMENT 01  |  EASY
EVEN / ODD PARITY — SENDER AND RECEIVER
=========================================================================
```

**What it does:** Encode a binary word with even/odd parity; check the received word and recover its data.

**Remember:** Count 1s → append parity bit → verify parity at receiver.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[101], encoded[102], received[102], scheme;
    int ones = 0, length;

    printf("Data: ");
    scanf("%100s", data);
    printf("Scheme (E/O): ");
    scanf(" %c", &scheme);
    if (scheme != 'E' && scheme != 'O') return 1;

    length = strlen(data);
    for (int i = 0; i < length; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        ones += data[i] == '1';
    }
    strcpy(encoded, data);
    encoded[length] = '0' + (ones + (scheme == 'O')) % 2;
    encoded[length + 1] = '\0';
    printf("Parity bit: %c\nEncoded: %s\n", encoded[length], encoded);

    printf("Received: ");
    scanf("%101s", received);
    if ((int)strlen(received) != length + 1) return 1;
    ones = 0;
    for (int i = 0; received[i]; i++) {
        if (received[i] != '0' && received[i] != '1') return 1;
        ones += received[i] == '1';
    }
    printf("Status: %s\n", ones % 2 == (scheme == 'O') ? "NO ERROR DETECTED" : "ERROR DETECTED");
    received[length] = '\0';
    printf("Extracted data: %s\n", received);
    return 0;
}
```

**INPUT — type these three values when prompted:**

```text
1011001
E
10110010
```

**EXPECTED OUTPUT:**

```text
Data: 1011001
Scheme (E/O): E
Parity bit: 0
Encoded: 10110010
Received: 10110010
Status: NO ERROR DETECTED
Extracted data: 1011001
```

**Why:** `1011001` contains four `1`s, so even parity appends `0`. The receiver detects no parity violation.

**To demonstrate an error:** Run again with the same data and `E`, but type `10100010` as the received word. The result becomes `Status: ERROR DETECTED` because one bit changed.



---


```text
=========================================================================
LAB 05  |  EXPERIMENT 02  |  EASY
PARITY WITH SCHEME INSIDE THE MESSAGE
=========================================================================
```

**What it does:** Send E/O along with the encoded word so the receiver extracts the scheme itself.

**Remember:** Transmit scheme + data + parity bit → extract and verify.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[101], sent[103], received[103], scheme;
    int length, ones = 0;

    printf("Data: ");
    scanf("%100s", data);
    printf("Scheme (E/O): ");
    scanf(" %c", &scheme);
    if (scheme != 'E' && scheme != 'O') return 1;

    length = strlen(data);
    for (int i = 0; i < length; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        ones += data[i] == '1';
    }
    sent[0] = scheme;
    strcpy(sent + 1, data);
    sent[length + 1] = '0' + (ones + (scheme == 'O')) % 2;
    sent[length + 2] = '\0';
    printf("Transmitted: %s\n", sent);

    printf("Received: ");
    scanf("%102s", received);
    if ((int)strlen(received) != length + 2 ||
        (received[0] != 'E' && received[0] != 'O')) return 1;
    ones = 0;
    for (int i = 1; received[i]; i++) {
        if (received[i] != '0' && received[i] != '1') return 1;
        ones += received[i] == '1';
    }
    printf("Extracted scheme: %c\n", received[0]);
    printf("Status: %s\n", ones % 2 == (received[0] == 'O') ? "NO ERROR DETECTED" : "ERROR DETECTED");
    received[length + 1] = '\0';
    printf("Extracted data: %s\n", received + 1);
    return 0;
}
```

**INPUT — type these values when prompted:**

```text
1011001
O
O10110011
```

**EXPECTED OUTPUT:**

```text
Data: 1011001
Scheme (E/O): O
Transmitted: O10110011
Received: O10110011
Extracted scheme: O
Status: NO ERROR DETECTED
Extracted data: 1011001
```

**Why:** `O` is transmitted before the bits. Four data `1`s require an appended `1` to make the total odd, and the receiver reads the scheme from the message.

**To demonstrate an error:** Type `O10100011` as the received word instead. The result becomes `Status: ERROR DETECTED`.



---


```text
=========================================================================
LAB 06  |  EXPERIMENT 01  |  MEDIUM
4B/5B BLOCK CODING — EVEN PARITY
=========================================================================
```

**What it does:** Encode four binary bits into five using even parity; validate and decode.

**Remember:** Four data bits + one parity bit. Detects single-bit flips.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[6], encoded[6], received[6];
    int ones = 0;

    printf("4-bit data: ");
    scanf("%5s", data);
    if (strlen(data) != 4) return 1;
    for (int i = 0; i < 4; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        ones += data[i] == '1';
    }

    strcpy(encoded, data);
    encoded[4] = '0' + ones % 2;
    encoded[5] = '\0';
    printf("5-bit codeword: %s\n", encoded);

    printf("Received codeword: ");
    scanf("%5s", received);
    if (strlen(received) != 5) return 1;
    ones = 0;
    for (int i = 0; i < 5; i++) {
        if (received[i] != '0' && received[i] != '1') return 1;
        ones += received[i] == '1';
    }
    if (ones % 2) {
        printf("Invalid codeword\n");
        return 0;
    }
    received[4] = '\0';
    printf("Valid codeword\nDecoded data: %s\n", received);
    printf("Matches input: %s\n", strcmp(data, received) ? "NO" : "YES");
    return 0;
}
```

**INPUT — type these values when prompted:**

```text
1011
10111
```

**EXPECTED OUTPUT:**

```text
4-bit data: 1011
5-bit codeword: 10111
Received codeword: 10111
Valid codeword
Decoded data: 1011
Matches input: YES
```

**Why:** `1011` has three `1`s, so adding parity bit `1` makes the five-bit word even-parity.

**To demonstrate an error:** Run again, receiving `10110` instead. The program prints `Invalid codeword`.



---


```text
=========================================================================
LAB 06  |  EXPERIMENT 02  |  MEDIUM
4B/5B BLOCK CODING — PDF LOOKUP TABLE
=========================================================================
```

**What it does:** Encode/decode using the exact 4B/5B mapping table supplied in the Lab 6 PDF.

**Remember:** Use when asked for the given 4B/5B table, not the parity-based mapping.

```c
#include <stdio.h>
#include <string.h>

int main() {
    const char *mapping[16] = {
        "11110", "01001", "10100", "10101",
        "01010", "01011", "01110", "01111",
        "10010", "10011", "10110", "10111",
        "11010", "11011", "11100", "11101"
    };
    char data[6], received[6], decoded[5];
    int index = 0, found = -1;

    printf("4-bit data: ");
    scanf("%5s", data);
    if (strlen(data) != 4) return 1;
    for (int i = 0; i < 4; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        index = index * 2 + data[i] - '0';
    }
    printf("Encoded: %s\n", mapping[index]);

    printf("Received: ");
    scanf("%5s", received);
    if (strlen(received) != 5) return 1;
    for (int i = 0; i < 16; i++)
        if (!strcmp(received, mapping[i])) found = i;
    if (found < 0) {
        printf("Invalid codeword\n");
        return 0;
    }

    for (int i = 0; i < 4; i++)
        decoded[i] = '0' + ((found >> (3 - i)) & 1);
    decoded[4] = '\0';
    printf("Valid codeword\nDecoded: %s\n", decoded);
    printf("Matches input: %s\n", strcmp(data, decoded) ? "NO" : "YES");
    return 0;
}
```

**INPUT — type these values when prompted:**

```text
0101
01011
```

**EXPECTED OUTPUT:**

```text
4-bit data: 0101
Encoded: 01011
Received: 01011
Valid codeword
Decoded: 0101
Matches input: YES
```

**Why:** The PDF lookup table maps `0101` to `01011`. The receiver looks for that codeword in the same table.

**To demonstrate an invalid codeword:** Enter `00000` as the received word. The program prints `Invalid codeword`.



---


```text
=========================================================================
LAB 06  |  EXPERIMENT 03  |  EASY
4B/5B CHECKER — VALIDITY AND MATCH
=========================================================================
```

**What it does:** Check a given five-bit parity codeword and compare its decoded data with the input.

**Remember:** Use with parity-based 4B/5B, not the supplied lookup table.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[6], codeword[6];
    int ones = 0;

    printf("4-bit dataword: ");
    scanf("%5s", data);
    printf("5-bit codeword: ");
    scanf("%5s", codeword);
    if (strlen(data) != 4 || strlen(codeword) != 5) return 1;

    for (int i = 0; i < 5; i++) {
        if (codeword[i] != '0' && codeword[i] != '1') return 1;
        ones += codeword[i] == '1';
    }
    for (int i = 0; i < 4; i++)
        if (data[i] != '0' && data[i] != '1') return 1;

    printf("Validity: %s\n", ones % 2 ? "INVALID" : "VALID");
    codeword[4] = '\0';
    if (ones % 2 == 0) printf("Extracted data: %s\n", codeword);
    printf("Matches input: %s\n", ones % 2 == 0 && !strcmp(data, codeword) ? "YES" : "NO");
    return 0;
}
```

**INPUT — type these values when prompted:**

```text
0101
01010
```

**EXPECTED OUTPUT:**

```text
4-bit dataword: 0101
5-bit codeword: 01010
Validity: VALID
Extracted data: 0101
Matches input: YES
```

**Why:** `01010` contains two `1`s and ends in parity bit `0`. It is valid under the even-parity coding scheme.

**To demonstrate invalid parity:** Use codeword `01011`. The output becomes `Validity: INVALID` and `Matches input: NO`.



---


```text
=========================================================================
LAB 07  |  EXPERIMENT 01  |  MEDIUM
AMI + B8ZS — SCRAMBLING AND RECOVERY
=========================================================================
```

**What it does:** Generate AMI and B8ZS signals and recover the original data.

**Remember:** AMI alternates +/− for 1; B8ZS replaces eight consecutive 0s.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], ami[201], b8zs[201], recovered[201], last = '-';
    int length;

    printf("Binary data: ");
    scanf("%200s", data);
    length = strlen(data);
    for (int i = 0; i < length; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        if (data[i] == '1') last = last == '+' ? '-' : '+';
        ami[i] = data[i] == '1' ? last : '0';
    }
    ami[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (i + 8 <= length && !strncmp(data + i, "00000000", 8)) {
            memcpy(b8zs + i, last == '+' ? "000+-0-+" : "000-+0+-", 8);
            i += 7;
        } else if (data[i] == '1') {
            last = last == '+' ? '-' : '+';
            b8zs[i] = last;
        } else b8zs[i] = '0';
    }
    b8zs[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (i + 8 <= length && !strncmp(b8zs + i,
            last == '+' ? "000+-0-+" : "000-+0+-", 8)) {
            memset(recovered + i, '0', 8);
            i += 7;
        } else {
            recovered[i] = b8zs[i] == '0' ? '0' : '1';
            if (b8zs[i] != '0') last = b8zs[i];
        }
    }
    recovered[length] = '\0';

    printf("AMI      : %s\nB8ZS     : %s\nRecovered: %s\n", ami, b8zs, recovered);
    printf("Verification: %s\n", strcmp(data, recovered) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type this binary sequence:**

```text
1000000001
```

**EXPECTED OUTPUT:**

```text
Binary data: 1000000001
AMI      : +00000000-
B8ZS     : +000+-0-+-
Recovered: 1000000001
Verification: SUCCESS
```

**Why:** The eight consecutive zeros trigger B8ZS substitution. `+` and `-` represent bipolar pulses, while `0` represents no pulse. Recovery restores the original zeros.



---


```text
=========================================================================
LAB 07  |  EXPERIMENT 02  |  HARD
HDB3 — SCRAMBLING AND RECOVERY
=========================================================================
```

**What it does:** Generate AMI and HDB3 and automatically verify recovered data.

**Remember:** Four 0s become B00V (even pulse count) or 000V (odd).

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], ami[201], hdb3[201], recovered[201], last = '-';
    int length, count = 0;

    printf("Binary data: ");
    scanf("%200s", data);
    length = strlen(data);
    for (int i = 0; i < length; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        if (data[i] == '1') last = last == '+' ? '-' : '+';
        ami[i] = data[i] == '1' ? last : '0';
    }
    ami[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (data[i] == '1') {
            last = last == '+' ? '-' : '+';
            hdb3[i] = last;
            count++;
        } else if (i + 4 <= length && !strncmp(data + i, "0000", 4)) {
            if (count % 2 == 0) {
                last = last == '+' ? '-' : '+';
                memcpy(hdb3 + i, last == '+' ? "+00+" : "-00-", 4);
            } else memcpy(hdb3 + i, last == '+' ? "000+" : "000-", 4);
            count = 0;
            i += 3;
        } else hdb3[i] = '0';
    }
    hdb3[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        recovered[i] = hdb3[i] == '0' ? '0' : '1';
        if (hdb3[i] != '0') {
            if (hdb3[i] == last) {
                recovered[i] = '0';
                if (i >= 3 && hdb3[i-3] == last &&
                    hdb3[i-2] == '0' && hdb3[i-1] == '0')
                    recovered[i-3] = '0';
            }
            last = hdb3[i];
        }
    }
    recovered[length] = '\0';

    printf("AMI      : %s\nHDB3     : %s\nRecovered: %s\n", ami, hdb3, recovered);
    printf("Verification: %s\n", strcmp(data, recovered) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type this binary sequence:**

```text
10000100001
```

**EXPECTED OUTPUT:**

```text
Binary data: 10000100001
AMI      : +0000-0000+
HDB3     : +000+-000-+
Recovered: 10000100001
Verification: SUCCESS
```

**Why:** Groups of four zeros are replaced using `000V` or `B00V`, depending on the nonzero pulse count since the last substitution; descrambling reverses them.



---


```text
=========================================================================
LAB 07  |  EXPERIMENT 03  |  HARD
COMPARISON — AMI, B8ZS AND HDB3
=========================================================================
```

**What it does:** Produce all signals, count zero runs/violations/pulses, and test both decoders.

**Remember:** Combined Lab 7 experiment: compare synchronization-related signal properties.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], ami[201], b8zs[201], hdb3[201];
    char decodedB[201], decodedH[201], last = '-';
    int length, zeros = 0, maxZeros = 0, count = 0;
    int bViolations = 0, hViolations = 0;
    int amiPulses = 0, bPulses = 0, hPulses = 0;

    printf("Binary data: ");
    scanf("%200s", data);
    length = strlen(data);
    for (int i = 0; i < length; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        zeros = data[i] == '0' ? zeros + 1 : 0;
        if (zeros > maxZeros) maxZeros = zeros;
        if (data[i] == '1') last = last == '+' ? '-' : '+';
        ami[i] = data[i] == '1' ? last : '0';
    }
    ami[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (i + 8 <= length && !strncmp(data + i, "00000000", 8)) {
            memcpy(b8zs + i, last == '+' ? "000+-0-+" : "000-+0+-", 8);
            bViolations += 2;
            i += 7;
        } else if (data[i] == '1') {
            last = last == '+' ? '-' : '+';
            b8zs[i] = last;
        } else b8zs[i] = '0';
    }
    b8zs[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (data[i] == '1') {
            last = last == '+' ? '-' : '+';
            hdb3[i] = last;
            count++;
        } else if (i + 4 <= length && !strncmp(data + i, "0000", 4)) {
            if (count % 2 == 0) {
                last = last == '+' ? '-' : '+';
                memcpy(hdb3 + i, last == '+' ? "+00+" : "-00-", 4);
            } else memcpy(hdb3 + i, last == '+' ? "000+" : "000-", 4);
            hViolations++;
            count = 0;
            i += 3;
        } else hdb3[i] = '0';
    }
    hdb3[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        if (i + 8 <= length && !strncmp(b8zs + i,
            last == '+' ? "000+-0-+" : "000-+0+-", 8)) {
            memset(decodedB + i, '0', 8);
            i += 7;
        } else {
            decodedB[i] = b8zs[i] == '0' ? '0' : '1';
            if (b8zs[i] != '0') last = b8zs[i];
        }
    }
    decodedB[length] = '\0';

    last = '-';
    for (int i = 0; i < length; i++) {
        decodedH[i] = hdb3[i] == '0' ? '0' : '1';
        if (hdb3[i] != '0') {
            if (hdb3[i] == last) {
                decodedH[i] = '0';
                if (i >= 3 && hdb3[i-3] == last &&
                    hdb3[i-2] == '0' && hdb3[i-1] == '0')
                    decodedH[i-3] = '0';
            }
            last = hdb3[i];
        }
    }
    decodedH[length] = '\0';

    for (int i = 0; i < length; i++) {
        amiPulses += ami[i] != '0';
        bPulses += b8zs[i] != '0';
        hPulses += hdb3[i] != '0';
    }
    printf("AMI : %s\nB8ZS: %s\nHDB3: %s\n", ami, b8zs, hdb3);
    printf("Longest zero run: %d\n", maxZeros);
    printf("Violations: B8ZS=%d HDB3=%d\n", bViolations, hViolations);
    printf("Signal elements: %d each\n", length);
    printf("Nonzero pulses: AMI=%d B8ZS=%d HDB3=%d\n", amiPulses, bPulses, hPulses);
    printf("B8ZS recovered: %s (%s)\n", decodedB, strcmp(data, decodedB) ? "FAIL" : "SUCCESS");
    printf("HDB3 recovered: %s (%s)\n", decodedH, strcmp(data, decodedH) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type this binary sequence:**

```text
1000000001
```

**EXPECTED OUTPUT:**

```text
Binary data: 1000000001
AMI : +00000000-
B8ZS: +000+-0-+-
HDB3: +000+-00-+
Longest zero run: 8
Violations: B8ZS=2 HDB3=2
Signal elements: 10 each
Nonzero pulses: AMI=2 B8ZS=6 HDB3=5
B8ZS recovered: 1000000001 (SUCCESS)
HDB3 recovered: 1000000001 (SUCCESS)
```

**Why:** The source contains an eight-zero run. B8ZS and HDB3 insert nonzero pulses to help clock synchronization; both decoders recover the original input. The violation and pulse counts are for this particular input.



---


```text
=========================================================================
LAB 08  |  EXPERIMENT 01  |  EASY
CHARACTER COUNT — FRAMING AND DEFRAMING
=========================================================================
```

**What it does:** Split text into count-prefixed frames and reconstruct the complete text.

**Remember:** Frame header stores the number of characters in that frame.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], frames[201][201], recovered[201] = "";
    int size, counts[201], total = 0;

    printf("Text data: ");
    fgets(data, sizeof(data), stdin);
    data[strcspn(data, "\n")] = '\0';
    printf("Frame size: ");
    scanf("%d", &size);
    if (size < 1 || size > 200) return 1;

    int length = strlen(data);
    for (int i = 0; i < length; i += size) {
        counts[total] = length - i < size ? length - i : size;
        strncpy(frames[total], data + i, counts[total]);
        frames[total][counts[total]] = '\0';
        printf("Frame %d: %d %s\n", total + 1, counts[total], frames[total]);
        total++;
    }

    for (int i = 0; i < total; i++)
        strncat(recovered, frames[i], counts[i]);
    printf("Recovered: %s\n", recovered);
    printf("Verification: %s\n", strcmp(data, recovered) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type the text, then frame size:**

```text
COMPUTERNETWORK
4
```

**EXPECTED OUTPUT:**

```text
Text data: COMPUTERNETWORK
Frame size: 4
Frame 1: 4 COMP
Frame 2: 4 UTER
Frame 3: 4 NETW
Frame 4: 3 ORK
Recovered: COMPUTERNETWORK
Verification: SUCCESS
```

**Why:** The 15 characters are split into blocks of 4, with the final block containing 3 characters. The receiver joins the stored frames in order.

**Note:** This input deliberately has **no space** between `COMPUTER` and `NETWORK`. You may also type spaces in the text; `fgets()` preserves them.



---


```text
=========================================================================
LAB 08  |  EXPERIMENT 02  |  EASY
BYTE STUFFING — STUFF AND DESTUFF
=========================================================================
```

**What it does:** Wrap data with FLAG F, escape F/E in data, and recover the original.

**Remember:** Insert ESC E before every data F or E; reverse at receiver.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], stuffed[405], recovered[201];
    int index = 0, count = 0;

    printf("Text data: ");
    fgets(data, sizeof(data), stdin);
    data[strcspn(data, "\n")] = '\0';

    stuffed[index++] = 'F';
    for (int i = 0; data[i]; i++) {
        if (data[i] == 'F' || data[i] == 'E')
            stuffed[index++] = 'E';
        stuffed[index++] = data[i];
    }
    stuffed[index++] = 'F';
    stuffed[index] = '\0';

    for (int i = 1; i < index - 1; i++) {
        if (stuffed[i] == 'E') i++;
        recovered[count++] = stuffed[i];
    }
    recovered[count] = '\0';

    printf("Stuffed: %s\nRecovered: %s\n", stuffed, recovered);
    printf("Verification: %s\n", strcmp(data, recovered) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type this text:**

```text
ABFCED
```

**EXPECTED OUTPUT:**

```text
Text data: ABFCED
Stuffed: FABEFCEEDF
Recovered: ABFCED
Verification: SUCCESS
```

**Why:** `F` is the flag and `E` is the escape. The sender escapes the data's `F` and `E`, adds boundary flags, and the receiver removes them.



---


```text
=========================================================================
LAB 08  |  EXPERIMENT 03  |  MEDIUM
BIT STUFFING — STUFF AND DESTUFF
=========================================================================
```

**What it does:** Use flag 01111110 and insert/remove a 0 after five consecutive 1s.

**Remember:** Sender stuffs 0 after five 1s; receiver removes it.

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[201], framed[500], recovered[201];
    int index = 8, ones = 0, count = 0;

    printf("Binary data: ");
    scanf("%200s", data);
    strcpy(framed, "01111110");
    for (int i = 0; data[i]; i++) {
        if (data[i] != '0' && data[i] != '1') return 1;
        framed[index++] = data[i];
        ones = data[i] == '1' ? ones + 1 : 0;
        if (ones == 5) {
            framed[index++] = '0';
            ones = 0;
        }
    }
    strcpy(framed + index, "01111110");

    ones = 0;
    for (int i = 8; i < index; i++) {
        recovered[count++] = framed[i];
        ones = framed[i] == '1' ? ones + 1 : 0;
        if (ones == 5) {
            if (i + 1 >= index || framed[i + 1] != '0') return 1;
            i++;
            ones = 0;
        }
    }
    recovered[count] = '\0';
    printf("Stuffed: %.*s\n", index - 8, framed + 8);
    printf("Framed: %s\nRecovered: %s\n", framed, recovered);
    printf("Verification: %s\n", strcmp(data, recovered) ? "FAIL" : "SUCCESS");
    return 0;
}
```

**INPUT — type this binary sequence:**

```text
1000001111101
```

**EXPECTED OUTPUT:**

```text
Binary data: 1000001111101
Stuffed: 10000011111001
Framed: 011111101000001111100101111110
Recovered: 1000001111101
Verification: SUCCESS
```

**Why:** The sender inserts `0` immediately after five consecutive `1`s in the data. It surrounds the stuffed bits with `01111110` flags; the receiver removes the stuffed zero and recovers the original bits.


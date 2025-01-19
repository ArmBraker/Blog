+++ 
draft = false
date = 2025-01-19T13:29:11+01:00
title = "HHC24 - Android"
description = "HHC24 - Android challange for reverse engineering android application and decrypt secrets."
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "Android",
    "Encryption",
    "ReverseEngineering"

]
externalLink = ""
series = []
+++

## Bronz 🥉

- Elf give us hint to use following two tools: APK Tool, JADX. So, lets setup the enviroment...
  - JADX: <https://github.com/skylot/jadx>
  - APK Tool: <https://github.com/iBotPeaches/Apktool/releases>

- Now we are able to decompile the APK file.
- There is database helper class which is handling the sqlLite3 database file:

```Java
private static final String DATABASE_NAME = "naughtynicelist.db";
```

- This class also contains the commans for inserting child with item into "NormalList" table:

```SQL
db.execSQL("INSERT INTO NormalList (Item) VALUES ('Carlos, Madrid, Spain');");
```

- After analyzes of decompiled files, we can figure out that the DB is build dynamically when APK is executed. Therefore we have 2 options how to solve the challange:
    1. Execute the APK and debug it with ADB or excract the DB file from application directory in AVD or phone.
    2. Analyze the source code and check if there is a function which will remore the child from list.
- As usual, I opted with option 2. 😂
- I found that there is also another class "MainActivity"
This class lots of functionality like building the GUI and button activities, **however** there are also another SQL statements.
- I found the one I was looking for:

```SQL
Cursor cursor = sQLiteDatabase.rawQuery("SELECT Item FROM NormalList WHERE Item NOT LIKE '%Ellie%'", null);
```

Now we know that Ellie will not get presents! 🎁

## Gold 🥇

- The Gold challange contains 2 smaller "challanges":

  - **Format**: We know that this APK is packaged with another format from ```Elf hint```:

     *"So yeah, have you heard about this new Android app format? Want to convert it to an APK file?"*
    The format was ```.aab```

    There is a GitHub repo: <https://github.com/HackJJ/apk-sherlock/blob/main/aab2apk.md>

  - **Encryption and Obfuscation**:

    *"Obfuscated and encrypted? Hmph. Shame you can't just run strings on the file."*

### Exctracting APK

- By following the instuction from GitHub. I extracted the APK from .aab:

```bash
Archive:  .\SantaSwipe.zip
 extracting: .\SantaSwipeSecure\/toc.pb
 extracting: .\SantaSwipeSecure\/universal.apk
```

- Analyzing the source code in JADX, I noticed that there is AES encryption with following parameters:
  - AES
  - GCM mode no padding

- Encryption parameters are listed in ```res/values/strings.xml```:

```xml
<string name="ek">rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw=</string> 
<string name="iv">Q2hlY2tNYXRlcml4</string>
```

- Using JADX I found in the source code DB statements which are encrypted:

- First statement:

```Java
db.execSQL(decryptData("IVrt+9Zct4oUePZeQqFwyhBix8cSCIxtsa+lJZkMNpNFBgoHeJlwp73l2oyEh1Y6AfqnfH7gcU9Yfov6u70cUA2/OwcxVt7Ubdn0UD2kImNsclEQ9M8PpnevBX3mXlW2QnH8+Q+SC7JaMUc9CIvxB2HYQG2JujQf6skpVaPAKGxfLqDj+2UyTAVLoeUlQjc18swZVtTQO7Zwe6sTCYlrw7GpFXCAuI6Ex29gfeVIeB7pK7M4kZGy3OIaFxfTdevCoTMwkoPvJuRupA6ybp36vmLLMXaAWsrDHRUbKfE6UKvGoC9d5vqmKeIO9elASuagxjBJ"));
```

- Second  statement:

```Java
  private final void insertInitialData(SQLiteDatabase db) {
        Iterator it = CollectionsKt.listOf((Object[]) new String[]{"L2HD1a45w7EtSN41J7kx/hRgPwR8lDBg9qUicgz1qhRgSg==", "IWna1u1qu/4LUNVrbpd8riZ+w9oZNN1sPRS2ujQpMqAAt114Yw==", "MWfO0+M1t5IvQtN2ad9w3hp81sYQIIaX6veq03bnk6I4H/1n89gW", "LmHJ164506skXdh3K9MZ/BBiw90TRO2mD0Hp9Nuoxu4ghx5/WQ==", "J2XF2645xKciX9RgK9MR+wZ60NIbKIsOTRSHP0jkBJPaF0djlqbc", "LGfJ0q451qslWt14aZd8rjtr1ZMtJItIvrKk8RRQWh2U6bQSEdPga59XDQ==", "LWTBzOt4u/4KXt99aJ18riBgy8cSJcpvtrKnM4IEsMDr9AwtlSJW+S/jdoHvXA==", "I2HM3+w1t4opQ953c5x8rjZvzNITIEFFaMC3bP4bW/FptwSEIzo=", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "O33D06452K0nWtA1J7kx/hRgtZNzI71BDpxorJ6mxImw/A==", "L2na0+M1t58yWdR3dN9wyQdrx9ASZFBP13TAtgvjKYVFW9J6rA==", "KmnGya451bs0Xdh3K9MX6wdjw90OdugnqIwKHDmVzoF7PjkQJQ==", "IWDN1K451bsvW9h3YN9wzR1nzNLx7AaHK857pwQ3GlLQ2i+e", "I2XByK451L8vQ941J7Y39wV6pk1FpbJdc3QZRlpVuVIsjQ==", "JGnc0+94u/4FUMJ4ZZ8x4BZvjpM6LphLvL+vfIsRMj38FLu+GyCyUdPJ2g==", "I2TNwq452bsxEeh2dZh8riBd4zTtytJ89Bxu9NEok+aDWmI=", "J2TN1OM1t5MpQtJ2cN9w3AB90doWBr/HMmnvw088K+A8OYyeqg==", "Ln3L26452rcqUN81J7ok7xl32UJTvrQCKVSd6Z+tiX2Hmg==", "Lm3B1uM1t5wjWMNsc99wwhBsw90YLyjPYh2rLfMJ1C4VX2kQBpE=", "KGfG2/E1t40yXtJyb5w841ku8cQSJY9K1bwgTiInx6cYV5Ui/DM+sA==", "L23BlqJK/78oVtl4bt9wzR1nzNKBr23V1KlRgUeyT5PagT9r", "MWnFz+d1u/4EVMN3K9MD+Rx62NYFLYtKu34IoIzUoRVftPjhSrdCxro=", "LGnM0+M1t5UvVMc1J6Y7/BRnzNY2Zug0wYGSAMBLA1AcU9yx", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2Hb0uM1t5AnWMN2ZZp8rj5rzMoWa/+6xCGsNnHvb00IU4Fr3Q==", "JmnG0+d1u/4JQt12K9Me4Qd5w8o6jn3t2IdQwSuXYx7u//Cv", "I2bBzuM1t5MzXNN4bt9wxxtqy9JRetksUopAr2UrK5xAUHDW", "JmHN3e01t5MjSdh6aNMT5wF3jpM6JJJNvLOoRJgv4sLKLQau2YBLzokQ", "LmHG3uM1t50pQdR3b5I36xsigvcSL4dFrbfU9Cqvl5I/xLcz0XU03pLK", "I2bc1ew1t4gvVN93Zt9wzwB91sEeIIGZ9QAB6FA50LlDQiHup1s=", "MWna264537sqQth3bJp8rjNnzN8WL47gYBHKOnMA/05AZVXhUewu", "KGfa3ec1t5IvXNA1J6M1/ACz8Z0iiXnvwFsEg5jrkLGq", "JXrJ2ec1t5siWN97coE35lku8dAYNYZFsbi05FzIyksNODX0NjFqyvVn", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t5QpWdB3aZYj7AB8xZ9XEoVRq7TgFosbO6NUOYiKnRWuNaKuxMtDzqVA7Q==", "LGfa26451rM1RdRrY5I9olVAx8cfJJhIvrKkJJm3opYYxGiM9SQhFp8qlek=", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK451rMrUN81J7k//BFvzGO14j2tV1Au4CjIir5oH4s=", "L2na0+c1t44nQ9hqK9MW/BRgwdazin6FJqgBFVKkiYZE/Oku", "NmfFlqJa/7clUNZ2K9MF3TR3qVJdK8qDeVx+ZJ0MHnNU", "Km3E3+x4u/4WQ9B+cpZ8rjZ0x9AfYbhBr6miO4QKWNrxWDpCUMmjm7xYtCdc6A==", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t4knQ8J4cN9w3hpiw90TTjWQOk7MWqw/XG0TO0AeQA==", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4KWMJ7aJ18riVh0McCJotIhSM4dPg694vcV/24WexvGQ==", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfNh3dJh8rjdrztIFNJmu2xg+J6QSEiIx6k9QIl+a", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t40pV9h4K9MS+xlpw8EeIHcUEuddwvPOGYIPlGbXuns=", "JmHF0/Zr/vJmZdNwa5oj51ku5dYYM41Nvhg6ySTDiTI5vv18a6J6y14=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452r80QtRwa581olVI0NIZIo+b1vqa1HqNcwPKRRUCkA2e", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmesN4bJwnolVezd8WL47o08C8QCvrmNi28s6uzbkB", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmcN9yZoExolVa18EcJJMyCPdCGbm3ITUv4dWqaqTJ", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451L8vQ941J7Y39wV6CD4JBxN6y16SruzezWKPkA==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K453q0yUN97cp98riF70NgSOCtbDxARU2xtlLgyGO7yaRU=", "Ln3LlqJb5as1QtR1dN9wzBBixdoCLK9D8qtP2JBdQm86rVaciwM=", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2TNwuN386wjHZFVcos14xdh18EQbcpoqqSlOo8GJ7JSkODr7+fBPtiGNds2i5nCLw==", "KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8=", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya45xL8oRdh4YJx8rjZmy98Sl+3RcG/g8aeHWMyfJPImOg==", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "L2nc3+01t5wpVt5txFJ8rjZhztwaI4NFdKbrVcU/JGUSzKJm3q3PSQ==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJb4q0nX505VJwl+h0u6dwFJIuiXi2lY6YPZ4CjViAKRpaC", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJK9rBmd8N4aZA5/RZhjpMiEqvWVxuq2sU0ngeuJh5Bmivp", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5wnX9Z4a5wi61ku690TKIvYZgzmW1HQgG0OvS+GZrqX", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t5kjX9RvZt9w3QJn1skSM4ZFsbjUNmfsNMnhkq0GBwNdRuQQ", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJR+Ks1Rd53K9MF3TQJIJtqQwmr4zdVTQ9laNPM", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJV+K1mcN9+Yp81/Vku9+A2aAwUTns96j8eMWk7TUtSxw==", "J2zf2/B95PJmf9RuJ7k1/AZr259XFLll5xvRfxu/YDOTFR460RwK+Q=="}).iterator();
        while (it.hasNext()) {
            db.execSQL("INSERT INTO NormalList (Item) VALUES ('" + StringsKt.trim((CharSequence) it.next()).toString() + "');");
        }
    }
```

- Using Python for data decryption:

```python
import base64
from Crypto.Cipher import AES
from Crypto.Util import Padding

# Base64 encoded IV and Encryption Key from strings.xml
base64_iv = "Q2hlY2tNYXRlcml4"
base64_key = "rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw="
base64_encrypted_data = "IVrt+9Zct4oUePZeQqFwyhBix8cSCIxtsa+lJZkMNpNFBgoHeJlwp73l2oyEh1Y6AfqnfH7gcU9Yfov6u70cUA2/OwcxVt7Ubdn0UD2kImNsclEQ9M8PpnevBX3mXlW2QnH8+Q+SC7JaMUc9CIvxB2HYQG2JujQf6skpVaPAKGxfLqDj+2UyTAVLoeUlQjc18swZVtTQO7Zwe6sTCYlrw7GpFXCAuI6Ex29gfeVIeB7pK7M4kZGy3OIaFxfTdevCoTMwkoPvJuRupA6ybp36vmLLMXaAWsrDHRUbKfE6UKvGoC9d5vqmKeIO9elASuagxjBJ"  # Replace with actual data

# Decode the Base64 strings
iv = base64.b64decode(base64_iv)
encryption_key = base64.b64decode(base64_key)
encrypted_data = base64.b64decode(base64_encrypted_data)

# AES-GCM Decryption
try:
    cipher = AES.new(encryption_key, AES.MODE_GCM, nonce=iv)
    decrypted_data = cipher.decrypt_and_verify(encrypted_data[:-16], encrypted_data[-16:])
    decrypted_text = decrypted_data.decode('utf-8')
    print("Decrypted Data:", decrypted_text)
except Exception as e:
    print("Decryption failed:", e)
```

- Decrypted statements:

```SQL
CREATE TRIGGER DeleteIfInsertedSpecificValue
    AFTER INSERT ON NormalList
    FOR EACH ROW
    BEGIN
        DELETE FROM NormalList WHERE Item = 'KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8=';
    END;
```

- According to above statement, we need to decrypt the Item once again to find deleted child:

```Joshua, Birmingham, United Kingdom```

- Decryption script for second SQL statement:

```python
import base64
from Crypto.Cipher import AES
from Crypto.Util import Padding

# Base64 encoded IV and Encryption Key from strings.xml
base64_iv = "Q2hlY2tNYXRlcml4"
base64_key = "rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw="
base64_encrypted_data = "L2HD1a45w7EtSN41J7kx/hRgPwR8lDBg9qUicgz1qhRgSg==", "IWna1u1qu/4LUNVrbpd8riZ+w9oZNN1sPRS2ujQpMqAAt114Yw==", "MWfO0+M1t5IvQtN2ad9w3hp81sYQIIaX6veq03bnk6I4H/1n89gW", "LmHJ164506skXdh3K9MZ/BBiw90TRO2mD0Hp9Nuoxu4ghx5/WQ==", "J2XF2645xKciX9RgK9MR+wZ60NIbKIsOTRSHP0jkBJPaF0djlqbc", "LGfJ0q451qslWt14aZd8rjtr1ZMtJItIvrKk8RRQWh2U6bQSEdPga59XDQ==", "LWTBzOt4u/4KXt99aJ18riBgy8cSJcpvtrKnM4IEsMDr9AwtlSJW+S/jdoHvXA==", "I2HM3+w1t4opQ953c5x8rjZvzNITIEFFaMC3bP4bW/FptwSEIzo=", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "O33D06452K0nWtA1J7kx/hRgtZNzI71BDpxorJ6mxImw/A==", "L2na0+M1t58yWdR3dN9wyQdrx9ASZFBP13TAtgvjKYVFW9J6rA==", "KmnGya451bs0Xdh3K9MX6wdjw90OdugnqIwKHDmVzoF7PjkQJQ==", "IWDN1K451bsvW9h3YN9wzR1nzNLx7AaHK857pwQ3GlLQ2i+e", "I2XByK451L8vQ941J7Y39wV6pk1FpbJdc3QZRlpVuVIsjQ==", "JGnc0+94u/4FUMJ4ZZ8x4BZvjpM6LphLvL+vfIsRMj38FLu+GyCyUdPJ2g==", "I2TNwq452bsxEeh2dZh8riBd4zTtytJ89Bxu9NEok+aDWmI=", "J2TN1OM1t5MpQtJ2cN9w3AB90doWBr/HMmnvw088K+A8OYyeqg==", "Ln3L26452rcqUN81J7ok7xl32UJTvrQCKVSd6Z+tiX2Hmg==", "Lm3B1uM1t5wjWMNsc99wwhBsw90YLyjPYh2rLfMJ1C4VX2kQBpE=", "KGfG2/E1t40yXtJyb5w841ku8cQSJY9K1bwgTiInx6cYV5Ui/DM+sA==", "L23BlqJK/78oVtl4bt9wzR1nzNKBr23V1KlRgUeyT5PagT9r", "MWnFz+d1u/4EVMN3K9MD+Rx62NYFLYtKu34IoIzUoRVftPjhSrdCxro=", "LGnM0+M1t5UvVMc1J6Y7/BRnzNY2Zug0wYGSAMBLA1AcU9yx", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2Hb0uM1t5AnWMN2ZZp8rj5rzMoWa/+6xCGsNnHvb00IU4Fr3Q==", "JmnG0+d1u/4JQt12K9Me4Qd5w8o6jn3t2IdQwSuXYx7u//Cv", "I2bBzuM1t5MzXNN4bt9wxxtqy9JRetksUopAr2UrK5xAUHDW", "JmHN3e01t5MjSdh6aNMT5wF3jpM6JJJNvLOoRJgv4sLKLQau2YBLzokQ", "LmHG3uM1t50pQdR3b5I36xsigvcSL4dFrbfU9Cqvl5I/xLcz0XU03pLK", "I2bc1ew1t4gvVN93Zt9wzwB91sEeIIGZ9QAB6FA50LlDQiHup1s=", "MWna264537sqQth3bJp8rjNnzN8WL47gYBHKOnMA/05AZVXhUewu", "KGfa3ec1t5IvXNA1J6M1/ACz8Z0iiXnvwFsEg5jrkLGq", "JXrJ2ec1t5siWN97coE35lku8dAYNYZFsbi05FzIyksNODX0NjFqyvVn", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t5QpWdB3aZYj7AB8xZ9XEoVRq7TgFosbO6NUOYiKnRWuNaKuxMtDzqVA7Q==", "LGfa26451rM1RdRrY5I9olVAx8cfJJhIvrKkJJm3opYYxGiM9SQhFp8qlek=", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK451rMrUN81J7k//BFvzGO14j2tV1Au4CjIir5oH4s=", "L2na0+c1t44nQ9hqK9MW/BRgwdazin6FJqgBFVKkiYZE/Oku", "NmfFlqJa/7clUNZ2K9MF3TR3qVJdK8qDeVx+ZJ0MHnNU", "Km3E3+x4u/4WQ9B+cpZ8rjZ0x9AfYbhBr6miO4QKWNrxWDpCUMmjm7xYtCdc6A==", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t4knQ8J4cN9w3hpiw90TTjWQOk7MWqw/XG0TO0AeQA==", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4KWMJ7aJ18riVh0McCJotIhSM4dPg694vcV/24WexvGQ==", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfNh3dJh8rjdrztIFNJmu2xg+J6QSEiIx6k9QIl+a", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t40pV9h4K9MS+xlpw8EeIHcUEuddwvPOGYIPlGbXuns=", "JmHF0/Zr/vJmZdNwa5oj51ku5dYYM41Nvhg6ySTDiTI5vv18a6J6y14=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452r80QtRwa581olVI0NIZIo+b1vqa1HqNcwPKRRUCkA2e", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmesN4bJwnolVezd8WL47o08C8QCvrmNi28s6uzbkB", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmcN9yZoExolVa18EcJJMyCPdCGbm3ITUv4dWqaqTJ", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451L8vQ941J7Y39wV6CD4JBxN6y16SruzezWKPkA==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K453q0yUN97cp98riF70NgSOCtbDxARU2xtlLgyGO7yaRU=", "Ln3LlqJb5as1QtR1dN9wzBBixdoCLK9D8qtP2JBdQm86rVaciwM=", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2TNwuN386wjHZFVcos14xdh18EQbcpoqqSlOo8GJ7JSkODr7+fBPtiGNds2i5nCLw==", "KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8=", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya45xL8oRdh4YJx8rjZmy98Sl+3RcG/g8aeHWMyfJPImOg==", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "L2nc3+01t5wpVt5txFJ8rjZhztwaI4NFdKbrVcU/JGUSzKJm3q3PSQ==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJb4q0nX505VJwl+h0u6dwFJIuiXi2lY6YPZ4CjViAKRpaC", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJK9rBmd8N4aZA5/RZhjpMiEqvWVxuq2sU0ngeuJh5Bmivp", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5wnX9Z4a5wi61ku690TKIvYZgzmW1HQgG0OvS+GZrqX", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t5kjX9RvZt9w3QJn1skSM4ZFsbjUNmfsNMnhkq0GBwNdRuQQ", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJR+Ks1Rd53K9MF3TQJIJtqQwmr4zdVTQ9laNPM", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJV+K1mcN9+Yp81/Vku9+A2aAwUTns96j8eMWk7TUtSxw==", "J2zf2/B95PJmf9RuJ7k1/AZr259XFLll5xvRfxu/YDOTFR460RwK+Q=="


# Decode the Base64 strings
iv = base64.b64decode(base64_iv)
encryption_key = base64.b64decode(base64_key)
encrypted_data = ""

for data in base64_encrypted_data: # As the data are now in the list, we need to iteratate in loop
    encrypted_data = base64.b64decode(data)
    # AES-GCM Decryption
    try:
        cipher = AES.new(encryption_key, AES.MODE_GCM, nonce=iv)
        decrypted_data = cipher.decrypt_and_verify(encrypted_data[:-16], encrypted_data[-16:])
        decrypted_text = decrypted_data.decode('utf-8')
        print("Decrypted Data:", decrypted_text)
    except Exception as e:
        print("Decryption failed:", e)

```

- According to SQL statement found in Source Code, these data (from above encrypted statement) are used to populate the DB:

{{% details "Click to Show/Hide Table" %}}

| Name      | City             | Country            |
|-----------|------------------|--------------------|
| Adam      | Bratislava       | Slovakia           |
| Adam      | Bratislava       | Slovakia           |
| Adam      | Bratislava       | Slovakia           |
| Ahmed     | Cairo            | Egypt              |
| Ahmed     | Cairo            | Egypt              |
| Ahmed     | Cairo            | Egypt              |
| Aiden     | Toronto          | Canada             |
| Aisha     | Nairobi          | Kenya              |
| Aisha     | Jakarta          | Indonesia          |
| Aisha     | Jakarta          | Indonesia          |
| Aisha     | Jakarta          | Indonesia          |
| Alex      | New York         | USA                |
| Alexandre | Luxembourg       | Luxembourg         |
| Alexandre | Brussels         | Belgium            |
| Alexandre | Brussels         | Belgium            |
| Ali       | Tunis            | Tunisia            |
| Ali       | Tunis            | Tunisia            |
| Ali       | Tunis            | Tunisia            |
| Alina     | Sofia            | Bulgaria           |
| Alina     | Bucharest        | Romania            |
| Alina     | Bucharest        | Romania            |
| Amir      | Cairo            | Egypt              |
| Ana       | Zagreb           | Croatia            |
| Ana       | Zagreb           | Croatia            |
| Ana       | Zagreb           | Croatia            |
| Anastasia | Saint Petersburg | Russia             |
| Anastasia | Saint Petersburg | Russia             |
| Anastasia | Saint Petersburg | Russia             |
| Andrei    | Chisinau         | Moldova            |
| Andrei    | Chisinau         | Moldova            |
| Andrei    | Chisinau         | Moldova            |
| Anita     | Mumbai           | India              |
| Anna      | Copenhagen       | Denmark            |
| Anna      | Copenhagen       | Denmark            |
| Anna      | Copenhagen       | Denmark            |
| Anton     | Vienna           | Austria            |
| Arjun     | Delhi            | India              |
| Arjun     | Bangalore        | India              |
| Arjun     | Delhi            | India              |
| Carlos    | Madrid           | Spain              |
| Catherine | Ottawa           | Canada             |
| Catherine | Ottawa           | Canada             |
| Catherine | Ottawa           | Canada             |
| Chen      | Beijing          | China              |
| Daniel    | Oslo             | Norway             |
| Daniel    | Caracas          | Venezuela          |
| Daniel    | Johannesburg     | South Africa       |
| Daniel    | Caracas          | Venezuela          |
| Daniel    | Johannesburg     | South Africa       |
| Daniel    | Caracas          | Venezuela          |
| Daniel    | Johannesburg     | South Africa       |
| David     | Johannesburg     | South Africa       |
| David     | Sydney           | Australia          |
| David     | Sydney           | Australia          |
| Diego     | Mexico City      | Mexico             |
| Diego     | Santo Domingo    | Dominican Republic |
| Diego     | Santo Domingo    | Dominican Republic |
| Diego     | Santo Domingo    | Dominican Republic |
| Dimitri   | Tbilisi          | Georgia            |
| Dimitri   | Kiev             | Ukraine            |
| Dimitri   | Kiev             | Ukraine            |
| Edwards   | New              |                    |
| Elena     | Moscow           | Russia             |
| Elias     | Helsinki         | Finland            |
| Elias     | Helsinki         | Finland            |
| Elias     | Helsinki         | Finland            |
| Elif      | Istanbul         | Turkey             |
| Elif      | Ankara           | Turkey             |
| Elif      | Ankara           | Turkey             |
| Emily     | Dublin           | Ireland            |
| Emily     | Dublin           | Ireland            |
| Emily     | Dublin           | Ireland            |
| Emma      | Sydney           | Australia          |
| Emma      | Auckland         | New Zealand        |
| Emma      | Auckland         | New Zealand        |
| Emma      | Auckland         | New Zealand        |
| Emmanuel  | Lagos            | Nigeria            |
| Emmanuel  | Lagos            | Nigeria            |
| Emmanuel  | Lagos            | Nigeria            |
| Enzo      | Marseille        | France             |
| Enzo      | Milan            | Italy              |
| Enzo      | Milan            | Italy              |
| Eva       | Budapest         | Hungary            |
| Eva       | Budapest         | Hungary            |
| Eva       | Budapest         | Hungary            |
| Fatima    | Casablanca       | Morocco            |
| Felix     | Munich           | Germany            |
| Felix     | Munich           | Germany            |
| Felix     | Munich           | Germany            |
| George    | Nairobi          | Kenya              |
| George    | Nairobi          | Kenya              |
| George    | Nairobi          | Kenya              |
| Grace     | Edinburgh        | Scotland           |
| Hans      | Berlin           | Germany            |
| Helena    | Prague           | Czech Republic     |
| Helena    | Vienna           | Austria            |
| Helena    | Vienna           | Austria            |
| Isabel    | Manila           | Philippines        |
| Isabel    | Manila           | Philippines        |
| Isabel    | Manila           | Philippines        |
| Isabella  | Rome             | Italy              |
| Isabella  | Rome             | Italy              |
| Isabella  | Rome             | Italy              |
| Isabella  | Rome             | Italy              |
| Jonas     | Stockholm        | Sweden             |
| Jonas     | Vilnius          | Lithuania          |
| Jonas     | Vilnius          | Lithuania          |
| Jorge     | Lima             | Peru               |
| Jose      | San José         | Costa Rica         |
| Jose      | San José         | Costa Rica         |
| Jose      | San José         | Costa Rica         |
| Joshua    | Birmingham       | United Kingdom     |
| Juan      | Quito            | Ecuador            |
| Juan      | Quito            | Ecuador            |
| Juan      | Quito            | Ecuador            |
| Julia     | Vienna           | Austria            |
| Julia     | Berlin           | Germany            |
| Julia     | Vienna           | Austria            |
| Julia     | Berlin           | Germany            |
| Julia     | Vienna           | Austria            |
| Julia     | Berlin           | Germany            |
| Kai       | Helsinki         | Finland            |
| Kai       | Helsinki         | Finland            |
| Kai       | Helsinki         | Finland            |
| Khadija   | Dakar            | Senegal            |
| Khadija   | Dakar            | Senegal            |
| Khadija   | Dakar            | Senegal            |
| Kim       | Seoul            | South Korea        |
| Kim       | Busan            | South Korea        |
| Kim       | Seoul            | South Korea        |
| Kim       | Seoul            | South Korea        |
| Laura     | Lima             | Peru               |
| Laura     | Madrid           | Spain              |
| Laura     | Lima             | Peru               |
| Laura     | Madrid           | Spain              |
| Laura     | Lima             | Peru               |
| Laura     | Madrid           | Spain              |
| Leila     | Beirut           | Lebanon            |
| Leila     | Algiers          | Algeria            |
| Leila     | Algiers          | Algeria            |
| Leila     | Algiers          | Algeria            |
| Leon      | Zagreb           | Croatia            |
| Leon      | Zagreb           | Croatia            |
| Leon      | Zagreb           | Croatia            |
| Liam      | Dublin           | Ireland            |
| Lina      | Cairo            | Egypt              |
| Lina      | Beirut           | Lebanon            |
| Lina      | Beirut           | Lebanon            |
| Linda     | Copenhagen       | Denmark            |
| Linda     | Reykjavik        | Iceland            |
| Linda     | Reykjavik        | Iceland            |
| Linda     | Reykjavik        | Iceland            |
| Luc       | Brussels         | Belgium            |
| Luc       | Paris            | France             |
| Luc       | Paris            | France             |
| Luca      | Milan            | Italy              |
| Lucia     | Barcelona        | Spain              |
| Lucia     | Barcelona        | Spain              |
| Lucia     | Barcelona        | Spain              |
| Lucia     | Barcelona        | Spain              |
| Luis      | Santiago         | Chile              |
| Luis      | Mexico City      | Mexico             |
| Luis      | Mexico City      | Mexico             |
| Maria     | Athens           | Greece             |
| Maria     | San Juan         | Puerto Rico        |
| Maria     | Lisbon           | Portugal           |
| Maria     | San Juan         | Puerto Rico        |
| Maria     | Lisbon           | Portugal           |
| Marie     | Paris            | France             |
| Marie     | Luxembourg       | Luxembourg         |
| Marie     | Luxembourg       | Luxembourg         |
| Marie     | Luxembourg       | Luxembourg         |
| Marta     | Warsaw           | Poland             |
| Marta     | Prague           | Czech Republic     |
| Marta     | Prague           | Czech Republic     |
| Mateo     | Buenos Aires     | Argentina          |
| Mateo     | Bogotá           | Colombia           |
| Mateo     | Buenos Aires     | Argentina          |
| Mateo     | Buenos Aires     | Argentina          |
| Mateusz   | Krakow           | Poland             |
| Mateusz   | Gdansk           | Poland             |
| Mateusz   | Gdansk           | Poland             |
| Mei       | Shanghai         | China              |
| Mei       | Taipei           | Taiwan             |
| Mei       | Taipei           | Taiwan             |
| Mia       | Tallinn          | Estonia            |
| Mia       | Tallinn          | Estonia            |
| Mia       | Tallinn          | Estonia            |
| Michael   | London           | United Kingdom     |
| Michael   | London           | United Kingdom     |
| Michal    | Bratislava       | Slovakia           |
| Michal    | Bratislava       | Slovakia           |
| Michal    | Bratislava       | Slovakia           |
| Miko      | Tokyo            | Japan              |
| Mustafa   | Ankara           | Turkey             |
| Mustafa   | Baghdad          | Iraq               |
| Mustafa   | Baghdad          | Iraq               |
| Nadia     | Kiev             | Ukraine            |
| Nikolai   | Minsk            | Belarus            |
| Nikolai   | Moscow           | Russia             |
| Nikolai   | Moscow           | Russia             |
| Nina      | Ljubljana        | Slovenia           |
| Nina      | Ljubljana        | Slovenia           |
| Nina      | Ljubljana        | Slovenia           |
| Noah      | Auckland         | New Zealand        |
| Nora      | Amsterdam        | Netherlands        |
| Nora      | Lagos            | Nigeria            |
| Nora      | Lagos            | Nigeria            |
| Nora      | Lagos            | Nigeria            |
| Olivia    | London           | United Kingdom     |
| Olivia    | Wellington       | New Zealand        |
| Olivia    | Wellington       | New Zealand        |
| Olivia    | Wellington       | New Zealand        |
| Omar      | Amman            | Jordan             |
| Omar      | Doha             | Qatar              |
| Omar      | Doha             | Qatar              |
| Omar      | Doha             | Qatar              |
| Peter     | Oslo             | Norway             |
| Peter     | Oslo             | Norway             |
| Peter     | Oslo             | Norway             |
| Rafael    | Lisbon           | Portugal           |
| Rafael    | Porto            | Portugal           |
| Rafael    | Porto            | Portugal           |
| Robert    | Glasgow          | Scotland           |
| Robert    | Glasgow          | Scotland           |
| Robert    | Glasgow          | Scotland           |
| Rosa      | Havana           | Cuba               |
| Rosa      | Havana           | Cuba               |
| Rosa      | Havana           | Cuba               |
| Samuel    | Bern             | Switzerland        |
| Samuel    | Vienna           | Austria            |
| Samuel    | Vienna           | Austria            |
| Samuel    | Vienna           | Austria            |
| Sara      | Helsinki         | Finland            |
| Sara      | Stockholm        | Sweden             |
| Sara      | Stockholm        | Sweden             |
| Simon     | Zurich           | Switzerland        |
| Simon     | Geneva           | Switzerland        |
| Simon     | Zurich           | Switzerland        |
| Sofia     | Lisbon           | Portugal           |
| Sofia     | Sofia            | Bulgaria           |
| Sofia     | Sofia            | Bulgaria           |
| Sofia     | Athens           | Greece             |
| Sofia     | Sofia            | Bulgaria           |
| Sofia     | Athens           | Greece             |
| Stefan    | Belgrade         | Serbia             |
| Stefan    | Belgrade         | Serbia             |
| Stefan    | Belgrade         | Serbia             |
| Stefan    | Belgrade         | Serbia             |
| Tom       | Chicago          | USA                |
| Tom       | San Francisco    | USA                |
| Tom       | Houston          | USA                |
| Tom       | Los Angeles      | USA                |
| Tomás     | Montevideo       | Uruguay            |
| Tomás     | Montevideo       | Uruguay            |
| Tomás     | Montevideo       | Uruguay            |
| Victor    | Bucharest        | Romania            |
| Victor    | Bucharest        | Romania            |
| Victor    | Bucharest        | Romania            |
| Victor    | Bucharest        | Romania            |
| Yasmine   | Casablanca       | Morocco            |
| Yasmine   | Casablanca       | Morocco            |
| Yasmine   | Casablanca       | Morocco            |
| Youssef   | Rabat            | Morocco            |
| Youssef   | Rabat            | Morocco            |
| Youssef   | Rabat            | Morocco            |
| Yuki      | Osaka            | Japan              |
| Zara      | Islamabad        | Pakistan           |
| Zara      | Islamabad        | Pakistan           |
| Zara      | Islamabad        | Pakistan           |

{{% /details %}}

- Lets submit the name of deleted child: ```Joshua, Birmingham, United Kingdom```

- ```Congratulations! You have completed the [Gold] Mobile Analysis challenge!```
- Now we know that Joshua will not get presents! 🎁

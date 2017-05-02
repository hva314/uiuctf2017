# uiuctf2017

### PapaRSA - 250
```python
#! /usr/bin/env python2

from Crypto.PublicKey import RSA

key = RSA.generate(4096, e=5)
msg = "this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it"
m = int(msg.encode("hex"), 16)
c = pow(m, key.e, key.n)

f = open("paparsa.txt", "w")
print >> f, "n = {}".format(key.n)
print >> f, "e = {}".format(key.e)
print >> f, "c = {}".format(c)
```
```
n = 805467500635696403604126524373650578882729068725582344971555936471728279008969317394226798274039587275908735628164913963756789131471531490012281262137708844664619411648776174742900969650281132608104486439462068493207388096754400356209191212924158917441463852311090597438686723680422989566039830705971272945580630621308622704812919416445637277433384864510484266136345300166188170847768250622904194100556098235897898548354386415341541887443486684297114240486341073977172459860420916964212739802004276614553755113124726331629822694410052832980560107812738167277181748569891715410067156205497753620739994002924247168259596220654379789860120944816884358006621854492232604827642867109476922149510767118658715534476782931763110787389666428593557178061972898056782926023179701767472969849999844288795597293792471883445525249025377326859655523448211020675915933552601140243332965620235850177872856558184848182439374292376522160931072677877590262080551636962148104050583711183119856867201924407132152091888936970437318064654447142605921825771487108398034919404885812834444299826080204996660391375038388918601615609593999711720104533648851576138805705999947802739408729788376315233147532770988216608571607302006681600662261521288802804512781133
e = 5
c = 321344338551168130701947757669249162791535374419225256466002854387287697945811581844875867845545337575193797350159207497966826027124926618458827324785590115214765980153475875175895244152171945352397663605222668892070894285036685408001675776259216704639659684767335997326195127379070104670798191048101430782486785148455557975065509824478935393935463232461294974471055239751453456270779997852527271795223623224696998441762750417393944955667837832299195592347653873362173157136283926817115042942127695355760288879165245940595259284499711202547364332122472169897570069773912201877037737474884548477516093671861643329899650704311880900221217905929830674467383904928054908475945599046498840246878554674443087280023564313470872269644230953001876937807402083390603760508851259383686896871724061532464374712413952574633098739843484563001012414107193262431117290853995664646176812763789444386869148000606985026530596652927567162641583951775993815884965569050328445927871220492529331846189285588168127051152438658813934744257031316581112434690871286836998078235766836485498780504037745116357109237384369621143931229920342036890494878183569174869563857473355851368119174926388706612127773670862261189669510108216517652686402185979222505401328291
```

The problem gave us a pretty legitimate RSA, except the fact that we know what the message looks like
```
this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it
```
Remeber the flag from babyRSA: `flag{c4n_w3_get_s0m3b0dy_t0_sm1th_some_c0pper_pls}` -> yup, CopperSmith baby!
Look up the CopperSmith method here: <https://www.wikiwand.com/en/Coppersmith_method> <br>
We have 
```
c = m^e (mod N)
-> m^e - c = 0 (mod N)
```
there, we can contruct our F(x)
```
f(x) = m^e - c
```
We already know part of message, so it would be 
```
f(x) = (M + m)^e - c
```
Coppersmith states that `Coppersmith’s algorithm can be used to find this integer solution x(0): x(0) < N^(1/e).`
However, if we use the whole part `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it` then `x` is pretty large in this case: `(2^8)^159 > N^(1/5)`. How about using only that XXX part as `m`. Seems promising.<br>
First, let's change all of those `X` into `\x00`. 
```python
sage: msg = msg.replace("X","\x00")
sage: msg
"this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: \x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it"
```
After that part of flag, we have 98 characters, add 1 null byte, we have 99. So if the flag is `m`, the original message is gonna be `M = msg + (2^8)^99*m`
From this point, using Sage we can find the roots for that equation of CopperSmith.
```python
sage: P.<x> = PolynomialRing(Zmod(n))
sage: f = (M + ((2^8)^99)*x)^5 - c
sage: f = f.monic()
sage: f.small_roots()
[]
```
Hmm, there is no roots found for this equation. The default epsilon sage has is `1/8`. Looks like we have to increase the upperbound of the algorithm. 
```python
sage: f.small_roots(epsilon=1/13)
[1248984295175060908103635259382502837476510996520290172164518365629374785269360163379181788573297776902028363990820288716208404068947393627909757]
```
There we go, that is `m`.
Here is the flag:
```python
sage: hex(int(m[0]))[2:-1].decode("hex")
'flag{bu7_0N_4_w3Dn3sdAy_iN_a_c4f3_i_waTcH3dD_17_6eg1n_aga1n}'
```
And here is the full message:
```python
sage: hex((int(m[0]^8)^99+M))[2:-1].decode("hex")
"this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: flag{bu7_0N_4_w3Dn3sdAy_iN_a_c4f3_i_waTcH3dD_17_6eg1n_aga1n}\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it"
```

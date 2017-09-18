# uiuctf2017

Official Write-up has been moved to [Blue's Little Blog](https://hva314.github.io/blog/2017/05/01/UIUCTF-2017-Crypto.html)

### BabyRSA - 200

```python 
#! /usr/bin/env python2

from Crypto.PublicKey import RSA

key = RSA.generate(4096, e=5)
msg = "welcome to uiuctf!\nyour super secret flag is: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
m = int(msg.encode("hex"), 16)
c = pow(m, key.e, key.n)

f = open("babyrsa.txt", "w")
print >> f, "n = {}".format(key.n)
print >> f, "e = {}".format(key.e)
print >> f, "c = {}".format(c)
```

```python
n = 826280450476795403105390383916395625701073920777162153138597185953056944510888027904354828464602421249363674719063026424044747076553321187265165775178889032794638105579799203345357910166892700405175658568627675449699540840288382597105404255643311670752496397923267416409538484199324051251779098290351314013642933189000153869540797043267546151497242578717464980825955180662199508957183411268811625401646070827084944007483568527240194185553478349118552388947992831458170444492412952312967110446929914832061366940165718329077289379496793520793044453012845571593091239615903167358140251268988719634075550032402744471298472559374963794796831888972573597223883502207025864412727194467531305956804869282127211781893423868568924921460804452906287133831167209340798856323714333552031073990953099946860260440120550744737264831895097569281340675979651355169393606387485601024283179141075124116079680183641040638005340147490312370291020282845417263785200481799143148652902589069064306494803532124234850362800892646823909347208346956741220877224626765444423081432186871792825772139369254830825377015531518313838382717867736340509229694011716101360463757629023320658921011843627332643744464724204771008866440681008984222122706436344770910544932757
e = 5
c = 199037898049081148054548566008626493558290050160287889209057083223407180156125399899465196611255722303390874101982934954388936179424024104549780651688160499201410108321518752502957346260593418668796624999582838387982430520095732090601546001755571395014548912727418182188910950322763678024849076083148030838828924108260083080562081253547377722180347372948445614953503124471116393560745613311509380885545728947236076476736881439654048388176520444109172092029548244462475513941506675855751026925250160078913809995564374674278235553349778352067191820570404315381746499936539482369231372882062307188454140330786512148310245052484671692280269741146507675933518321695623680547732771867757371698350343979932499637752314262246864787150534170586075473209768119198889190503283212208200005176410488476529948013610803040328568552414972234514746292014601094331465138374210925373263573292609023829742634966280579621843784216908520325876171463017051928049668240295956697023793952538148945070686999838223927548227156965116574566365108818752174755077045394837234760506722554542515056441166987424547451245495248956829984641868331576895415337336145024631773347254905002735757
```

First look at the code encryption, `e` is quite small for a large `n`, a 4096-bit key. Yet the message is not very long, 96 chars.
```
m^e ~ ((2^8)^96)^5 = 2^(8*96*5) = 2^3840 < 2^4096 ~ N
=> m^e < N
```
So, there is no way this m^e could exceed N
```python
sage: m = pow(c, 1/5)
sage: hex(int(m))[2:-1].decode('hex')
'welcome to uiuctf!\nyour super secret flag is: flag{c4n_w3_get_s0m3b0dy_t0_sm1th_some_c0pper_pls}'
```

<hr>

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

The problem gave us a pretty legitimate RSA, except the fact that we already knew what the message looks like
```
this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it
```
Remeber the flag from babyRSA: `flag{c4n_w3_get_s0m3b0dy_t0_sm1th_some_c0pper_pls}` -> yup, CopperSmith baby!
Look up the CopperSmith method here: <https://www.wikiwand.com/en/Coppersmith_method> <br>
We have:
```
c = m^e (mod N)
-> m^e - c = 0 (mod N)
```
From there, we can contruct our f(x)
```
f(x) = m^e - c
```
We already knew parts of the message, so f(x) would be 
```
f(x) = (M + m)^e - c
```
Coppersmith states that `Coppersmith’s algorithm can be used to find this integer solution x(0): x(0) < N^(1/e).`
However, if we use the whole part `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it` as `m`, the difference between the known and unknown, `m` is pretty large in this case: `(2^8)^159 >> N^(1/5)`. How about using only that XXX part as `m`. Seems promising.<br>
First, let's change all of those `X` into `\x00`. 
```python
sage: msg = msg.replace("X","\x00")
sage: msg
"this challenge was supposed to be babyrsa but i screwed up and now i have to redo the challenge.\nhopefully this challenge proves to be more worthy of 250 points compared to the 200 points i gave out for babyrsa :D :D :D\nyour super secret flag is: \x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\nyou know what i'm going to add an extra line here just to make your life miserable so deal with it"
```
Right after the "XXX", there are 98 other characters, 1 null byte, so we have 99. If the flag is `m`, the original message is gonna be `M = msg + (2^8)^99*m`.<br>
You can imagine: `9999955599999 = 9999900099999 + 555*(10^5`.<br>
From this point, using Sage we can find the roots for f(x).
```python
sage: msg = int(msg.encode("hex"), 16)
sage: P.<x> = PolynomialRing(Zmod(n))
sage: f = (msg + ((2^8)^99)*x)^5 - c
sage: f = f.monic()
sage: f.small_roots()
[]
```
Hmm, no root found. The default epsilon sage has is `1/8`. Looks like we have to increase the upperbound of the calculation. 
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
Pretty fun challenge. It's kinda mind-blowing when I realized the flag of babyRSA was actually a hint for this one :D 

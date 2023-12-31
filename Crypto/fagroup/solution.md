Κι άλλο RSA challenge. Yay . . .\
Αναλύοντας το `source.py` βλέπουμε ότι πρόκειται για κανονικό rsa με δύο διαφορές:
- Οι πρώτοι p, q είναι της μορφής `2*p1*p2*p3*p4*p5*p' + 1`, όπου `p1, p2, ..., p5` είναι πρώτοι μεταξύ 32-64 και `p'` είναι ένας τυχαίος μεγάλος πρώτος.
- Μας δίνεται ως επιπλέον πληροφορία ένας αριθμός `ord` για τον οποίο ξέρουμε ότι υπάρχει `r` τέτοιο ώστε `r^ord % n = 1`

Αν και δε μας δίνεται πως βρέθηκε το `ord`(χεχεχε) θα δούμε ότι με λίγες γνώσεις θεωρίας ομάδων ή/και του πως δουλεύει το RSA μπορούμε να παραγοντοποιήσουμε το `n` και κατ'επέκταση να αποκρυπτογραφήσουμε το flag.

Στο RSA δουλεύουμε σε μια ομάδα τάξης `φ(n)` όπου `φ()` η συνάρτηση του Euler. Αυτό σημαίνει ότι κάθε στοιχείο θα έχει τάξη κάποιον διαιρέτη του `φ(n)`, όπου τάξη του στοιχείου εννοούμε `στοιχείο^τάξη = 1 (mod n)`. Όμως αφού `r^ord = 1 (mod n)`, το `ord` που μας δίνεται θα είναι η τάξη κάποιου στοιχείου(που δεν το ξέρουμε αλλά δεν πειράζει) και συνεπώς θα είναι διαιρέτης του `φ(n)`!\
\
Επιπλέον, από τη γνωστή δομή των πρώτων έχουμε `φ(n) = (p - 1)*(q - 1) = (2*p1*p2*p3*p4*p5*p' + 1 - 1)*(2*q1*q2*q3*q4*q5*q' + 1 - 1) = 4*p1*p2*p3*p4*p5*q1*q2*q3*q4*q5*p'*q'`.\
\
Το `ord` θα διαιρεί όλο αυτό το γινόμενο και καθώς είναι πρώτος(έυκολα το ελέγχουμε με την isPrime(ord) ή οποιοδήποτε άλλο τρόπο) καταλαβαίνουμε ότι πρόκειται είτε για το `p'` είτε για το `q'`. 

Όμως, έχοντας το `p'`(ή αντίστοιχα το `q'`, δεν έχει σημασία), για να βρούμε το `p`, αρκεί μονάχα να βρούμε τα `p1, p2, p3, p4, p5` τα οποία είναι 5 πρώτοι στο εύρος 32-64. Σε αυτό το διάστημα οι μοναδικοι πρώτοι είναι οι `37, 41, 43, 47, 53, 59, 61`, οι οποίοι αριθμούν 7 στο πλήθος οπότε μπορούμε εύκολα να δοκιμάσουμε όλους τους συνδυασμούς μέχρι να βρούμε το σωστό `p`! Ο έλεγχος ότι το βρήκαμε σωστά μπορεί να γινεί επειδή μόνο το σωστό `p` θα διαιρεί τέλεια το `n` αφήνοντας υπόλοιπο 0.


Solver:
```python
from Crypto.Util.number import *

n = 142829528763805699857542233635011565789709504905698135923529590694545444315326567845469043841603447461822622170686363460371930363927327236001683910150983816569562129963300305571745228528229257966994498448633379749850923149040629616332024377579262234025580455223012093875998423154894040364167887925859745336557
ord = 37707689409093918855854078482105300715753037083483679716333774382530063354970586797606668771374004054430403261559763120731236205265668441930414743
c = 62244765642124693925452218053588298373027026087050588231772935070949523914497511619635142296225679214938973985554763413186777367874133050710931536501033894674668584193797208425804831741205409729247732964757678520476265649979959855344878224979215085474804291389637148669566919945656645161660639132076060567493

ord = ord * 2 

primes32_64 = [37, 41, 43, 47, 53, 59, 61]

for p1 in primes32_64:
    for p2 in primes32_64:
        for p3 in primes32_64:
            for p4 in primes32_64:
                for p5 in primes32_64:
                    potential_p = ord * p1 * p2 * p3 * p4 * p5 + 1
                    if n % potential_p == 0:
                        p = potential_p
                        q = n//p
                        phi = (p - 1) * (q - 1)
                        d = pow(65537, -1, phi)
                        m = pow(c, d, n)
                        print(long_to_bytes(m))
                        exit()

```


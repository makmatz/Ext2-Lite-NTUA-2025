## Εργαστήριο Λειτουργικών Συστημάτων | ΣΗΜΜΥ ΕΜΠ 2025-26
# Σύστήματα Αχείων, ext2-lite

### Ομάδα oslab6
- **Μαντζώρος Γεράσιμος** (03122011)
- **Μωραΐτης Δημήτρης** (03122175)

----

# Ερωτήσεις για τον δίσκο του αρχείου fsdisk1.img

## 1. Προσθήκη `fdisk1.img` στο utopia.

Για να προσθέσουμε στην εικονική μηχανή utopia έναν επιπλέον δίσκο για την εικόνα `fdisk1.img`, προσθέτουμε την παράμετρο `- drive file=ext2-vdisks/fsdisk1.img,format=raw,if=virtio` στην εντολή εκκίνησης του QEMU μέσα στο αρχείο `utopia.sh`. Η συσκευή που προσθέσαμε εμφανίζεται στο utopia ως `/dev/vdb`.

## 2. Εύρεση μεγέθους δίσκου.

### Tools:

Εκτελόυμε την εντολή `lsblk /dev/vdb`. Στο τερματικό εμφανίζεται `SIZE=50M` που σημαίνει ότι ο δίσκος έχει μέγεθος 50 MB.

### Hexedit:

Εκτελώντας την εντολή `ls -lh fsdisk1.img` στον host βλέπουμε στο πεδίο για το μέγεθος του αρχείου την τιμή 50Μ.

## 3. Εύρεση τύπου ΣΑ.

### Tools:

Εκτελόυμε την εντολή `lsblk -f /dev/vdb`. Στα μεταδεδομένα εμπεριέχεται `FSTYPE=ext2` που σημαίνει ότι το ΣΑ είναι ext2.

### Hexedit:

Στο ext2, το superblock ξεκινά στο offset 1024 bytes. Το magic number βρίσκεται στο offset 0x38 του superblock. Εκτελούμε την εντολή `hexdump -s 1024 -n 64 fsdisk1.img`. Στα bytes 56 και 57 εμφανίζεται το magic number = 0xef53 που δηλώνει ότι το ΣΑ είναι τυπου ext.

## 4. Χρονοσφραγίδα δημιουργίας.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Filesystem created: Tue Dec 12 15:23:16 2023`. 

### Hexedit:

Δεν περιλαμβάνεται.

## 5. Χρονοσφραγίδα τελευταίας προσάρτησης.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last mount time: Tue Dec 12 15:23:16 2023`.

### Hexedit: 

Η χρονοσφραγίδα τελευταίας προσάρτησης βρίσκεται στο offset 0x2c του superblock. Εκτελούμε τις εντολές: `TS=$(hexdump -s $((1024+0x2C)) -n 4 -e '1/4 "%u"' /dev/vdb)`, `date -d "@$TS"`. Παίρνουμε `Tue Dec 12 03:23:16 PM UTC 2023`.

## 6. Μονοπάτι τελευταίας προσάρτησης.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last mounted on: /cslab-bunker`. 

### Hexedit: 

Το μονοπάτι βρίσκεται σε offset 0x88. Εκτελούμε την εντολή `hexdump -s $((1024+0x88)) -n 64 -C /dev/vdb`. Αποτέλεσμα:
```
00000488  2f 63 73 6c 61 62 2d 62  75 6e 6b 65 72 00 00 00  |/cslab-bunker...|
00000498  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

## 7. Χρονοσφραγίδα τελευταίας τροποποίησης.

### Tools: 

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last write time: Tue Dec 12 15:23:17 2023`.

### Hexedit: 

Η χρονοσφραγίδα τελευταίας τροποποίησης βρίσκεται στο offset 0x30 του superblock. Εκτελούμε τις εντολές: `TS=$(hexdump -s $((1024+0x30)) -n 4 -e '1/4 "%u"' /dev/vdb)`, `date -d "@$TS"`. Παίρνουμε `Tue Dec 12 03:23:17 PM UTC 2023`.

## 8. Ορισμός Block

Σε ένα σύστημα αρχείων το Block είναι η ελάχιστη μονάδα αποθήκευσης δεδομένων.

## 9. Block size

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Block size: 1024`. Άρα το Block size είναι 1 KB. 

### Hexedit:

Σε offset 0x18 βρίσκουμε την τιμή s_log_block_size = 0, με την εντολή `LBS=$(hexdump -s $((1024+0x18)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $LBS`. BS = `1024 << s_log_block_size` = 1024.

## 10. Ορισμός Inode

Σε ένα σύστημα αρχείων το Inode είναι είναι μια δομή μεταδεδομένων που περιγράφει ένα αρχείο ή έναν κατάλογο. To Inode περιέχει τις εξής πληροφορίες:

- Τύπος αρχείου (character device, block device, regular file, directory κλπ).
- Δικαιώματα πρόσβασης.
- Αναγνωριστικό χρήστη / group.
- Χρονοσφραγίδες.
- Αριθμός Hard Links του αρχείου.
- Δείκτες στα Blocks δεδομένων του αρχείου.

## 11. Inode size.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Inode size: 128`.### Hexedit:

### Hexedit:

Σε offset 0x58 βρίσκεται το inode size. Με την εντολή `hexdump -s $((1024+0x58)) -n 2 -e '1/2 "%u\n"' /dev/vdb` παίρνουμε 128.

## 12. Διαθέσιμα Blocks και Inodes.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Free blocks: 49552, Free inodes: 12810`.

### Hexedit:

Το free blocks βρίσκεται σε offset 0x0c και το free inodes στο 0x10. Εκτελούμε τις εντολές `hexdump -s $((1024+0x0C)) -n 4 -e '1/4 "free_blocks=%u\n"' /dev/vdb` και `hexdump -s $((1024+0x10)) -n 4 -e '1/4 "free_inodes=%u\n"' /dev/vdb` και παίρνουμε free_blocks=49552 και free_inodes=12810.

## 13. Ορισμός Superblock.

Σε ένα σύστημα αρχείων το superblock είναι μια κεντρική δομή μεταδεδομένων που περιγράφει τη συνολική οργάνωση και κατάστασή του. Κάποιες από τις πληροφορίες που περιέχει είναι οι εξής:

- Τύπος συστήματος αρχείων.
- Block / Inode size.
- Συνολικός αριθμός Blocks / Inodes.
- Χρονοσφραγίδες.
- Πληροφορίες για τα groups του ΣΑ.

## 14. Θέση Superblock.

Στο σύστημα αρχείων ext2, το superblock βρίσκεται στο offset 1024 bytes (1 KB) από την αρχή του filesystem.

## 15. Εφεδρικά αντίγραφα Superblock.

Τα εφεδρικά αντίγραφα του superblock στο ext2 υπάρχουν για λόγους αξιοπιστίας και ανάκτησης, ώστε σε περίπτωση καταστροφής του κύριου superblock να μπορεί να αποκατασταθεί το σύστημα αρχείων και να διατηρηθεί η πρόσβαση στα δεδομένα.

## 16. Θέσεις αντιγράγων του Superblock.

### Tools:

Εκτελώντας την εντολη `dumpe2fs /dev/vdb | grep -i superblock` λαμβάνουμε:
```
Primary superblock at 1, Group descriptors at 2-2
Backup superblock at 8193, Group descriptors at 8194-8194
Backup superblock at 16385, Group descriptors at 16386-16386
Backup superblock at 24577, Group descriptors at 24578-24578
Backup superblock at 32769, Group descriptors at 32770-32770
Backup superblock at 40961, Group descriptors at 40962-40962
Backup superblock at 49153, Group descriptors at 49154-49154
```

Άρα έχουμε αντίγραφα του Superblock στα Blocks 8193, 16385, 24577, 32769, 40961 και 49153 (δηλαδή, κάθε 8192 Blocks). Οι θέσεις αυτές συμπίπτουν με την αρχή των Block groups.

### Hexedit:

Μπορούμε να υποθέσουμε ότι βρίσκονται στην αρχή κάθε group.

## 17. Block groups.

Ένα block group στο σύστημα αρχείων ext2 είναι μια λογική υποδιαίρεση του filesystem, που χωρίζει τον δίσκο σε μικρότερες, ενότητες για καλύτερη οργάνωση και αξιοπιστία. Κάθε Block group, εκτός από τα Data Blocks, περιέχει και άλλες χρήσιμες πληροφορίες όπως:

- Αντίγραφο Superblock.
- Block bitmap.
- Inode bitmap.
- Inode table.
- Block group descriptor.

## 18. Πλήθος Block groups.

Σε ένα σύστημα αρχείων ext2, ο αριθμός των block groups δεν είναι σταθερός αλλά εξαρτάται από το συνολικό μέγεθος του filesystem και από το μέγεθος των blocks. Το filesystem χωρίζεται σε πολλαπλά block groups, τα οποία κατανέμονται σειριακά στον δίσκο. Ο αριθμός των block groups υπολογίζεται με τον παρακάτω τύπο:

$$
\text{Αριθμός Block Groups} =
\left\lceil
\frac{\text{Συνολικά Blocks του Filesystem}}
{\text{Blocks ανά Group}}
\right\rceil
$$

όπου:
- Συνολικά Blocks του Filesystem = (Μέγεθος Filesystem) / (Μέγεθος Block).
- Blocks ανά Group = σταθερός αριθμός που καθορίζεται κατά τη δημιουργία του filesystem και εξαρτάται από το μέγεθος του block.

## 19. Αριθμός Blocks groups στο `fdisk1.img`.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Block count: 51200, Blocks per group: 8192`. Από τον παραπάνω τύπο προκύπτει ότι έχουμε 7 groups.

### Hexedit:

Ο αριθμός των blocks βρίσκεται σε offset 0x04 και των blocks per group στο 0x20. Χρησιμοποιούμε τις εντολές `TOTAL=$(hexdump -s $((1024+0x04)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $TOTAL`, `BPG=$(hexdump -s $((1024+0x20)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $BPG` και `echo $(( (TOTAL + BPG - 1) / BPG ))` και παίρνουμε το αποτέλεσμα: 7.

## 20. Block group descriptors.

Ο block group descriptor στο ext2 είναι μια δομή μεταδεδομένων που περιγράφει τη διάταξη και την κατάσταση ενός block group, αποθηκεύοντας πληροφορίες για τη θέση των bitmaps, του inode table και τον αριθμό των ελεύθερων πόρων.

## 21. Εφεδρικά αντίγραφα Block group descriptors.

Τα εφεδρικά αντίγραφα των block group descriptors στο σύστημα αρχείων ext2 υπάρχουν για λόγους αξιοπιστίας και ανάκτησης, καθώς οι descriptors περιγράφουν τη θέση και την κατάσταση των βασικών δομών κάθε block group. Σε περίπτωση καταστροφής του κύριου πίνακα descriptors, τα εφεδρικά αντίγραφα επιτρέπουν την αποκατάσταση του filesystem και τη διατήρηση της πρόσβασης στα δεδομένα.

## 22. Θέση bgd αντίγραφων.

### Tools:

Εκτελώντας την εντολη `dumpe2fs /dev/vdb | grep -i descriptors` λαμβάνουμε:
```
Primary superblock at 1, Group descriptors at 2-2
Backup superblock at 8193, Group descriptors at 8194-8194
Backup superblock at 16385, Group descriptors at 16386-16386
Backup superblock at 24577, Group descriptors at 24578-24578
Backup superblock at 32769, Group descriptors at 32770-32770
Backup superblock at 40961, Group descriptors at 40962-40962
Backup superblock at 49153, Group descriptors at 49154-49154
```

Άρα τα εφεδρικά αντίγραφα των Block group descriptors βρίσκονται αμέσως μετά από τα εφεδρικά αντίγραφα των superblocks.

### Hexedit:

Γνωρίζουμε ότι βρίσκονται μετά από τα εφεδρικά αντίγραφα των superblocks.

## 23. Block / Inode bitmap.

Το block bitmap και το inode bitmap στο ext2 είναι δομές μεταδεδομένων που αποθηκεύουν σε μορφή bit την κατάσταση (ελεύθερο ή δεσμευμένο) των blocks και των inodes αντίστοιχα και βρίσκονται μέσα σε κάθε block group, με τις ακριβείς θέσεις τους να καθορίζονται από τους block group descriptors.

## 24. Inode tables.

Τα inode tables στο ext2 είναι περιοχές του δίσκου που αποθηκεύουν τις δομές inode και βρίσκονται μέσα σε κάθε block group, μετά τα bitmaps, με τη θέση τους να καθορίζεται από τους block group descriptors.

## 25. Inodes.

Κάθε inode στο ext2 περιέχει μεταδεδομένα ενός αρχείου, όπως δικαιώματα, μέγεθος, χρονικές σφραγίδες και δείκτες προς data blocks, και αποθηκεύεται μέσα στα inode tables που βρίσκονται σε κάθε block group του filesystem.

## 26. Αριθμός Blocks / Inodes.

### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Blocks per group: 8192, Inodes per group: 1832`.

### Hexedit:

To blocks per group βρίσκεται σε offset 0x20 και το inodes per group στο 0x28. Με τις εντολές `hexdump -s $((1024+0x20)) -n 4 -e '1/4 "blocks_per_group=%u\n"' /dev/vdb`, `hexdump -s $((1024+0x28)) -n 4 -e '1/4 "inodes_per_group=%u\n"' /dev/vdb` παίρνουμε `blocks_per_group=8192, inodes_per_group=1832`.

## 27. Εύρεση inode του `/dir/helloworld`.

### Tools:

Εκτελούμε την εντολη `debugfs -R "stat /dir2/helloworld" /dev/vdb`. Στην πρώτη γραμμή εμφανίζεται το inode που αντιστοιχεί το συγκεκριμένο αρχείο, `Inode: 9162`.

### Hexedit:

Με hexedit στο αρχείο `/dev/vdb` ψάχνουμε το string "helloworld" (68656c6c6f776f726c64). Υπάρχει μία μόνο φορά, άρα η τιμή 8 bytes πριν από αυτο αντιστοιχεί στο αντίστοιχο inode. Η τιμή αυτή είναι 0xca23 (LE) = 9162.

## 28. Εύρεση group του `/dir/helloworld`.

Τα inodes κατανέμονται σειριακά στα groups και έχουμε 1832 inodes per group. Αρα το 9162 βρίσκεται στο group 5.

## 29. Εύρεση Inode table του Inode 9162.

### Tools:

Το συγκεκριμένο Inode βρίσκεται, όπως είδαμε, στο group 5. Εκτελούμε την εντολή `dumpe2fs /dev/vdb | grep -A 5 "Group 5"`. Βλέπουμε `Inode table at 40965-41193`.

### Hexedit:

Στο Block 2 υπάρχει το group descriptor table. Σε offset 2048+(5×32)=2048+160=2208=0x8a0 βρίσκεται το group descriptor του group 6. Τα bytes 8 - 11 δείχνουν το block που ξεκινάει το inode table. Με hexedit βρίσκουμε 05 A0 (LE) -> 0xa005 = 40965.

## 30. Περιεχόμενα του Inode 9162.

### Tools:

Εκτελούμε την εντολή `debugfs -R "stat <9162>" /dev/vdb`. Αποτέλεσμα:

```
Inode: 9162   Type: regular    Mode:  0644   Flags: 0x0
Generation: 2739270588    Version: 0x00000001
User:     0   Group:     0   Size: 42
File ACL: 0
Links: 1   Blockcount: 2
Fragment:  Address: 0    Number: 0    Size: 0
ctime: 0x65787ae4 -- Tue Dec 12 15:23:16 2023
atime: 0x65787ae4 -- Tue Dec 12 15:23:16 2023
mtime: 0x65787ae4 -- Tue Dec 12 15:23:16 2023
BLOCKS:
(0):1025
TOTAL: 1
```

### Hexedit:

To inode είναι δεύτερο στο block. Η αρχή του table είναι στο 40965 * 1024 = 41,948,160. 128 bytes μετά την αρχή βρίσκεται το inode. ;Άρα στη θεση 0x2801480. Περιεχόμανο:

```
02801480   A4 81 00 00  2A 00 00 00  E4 7A 78 65  E4 7A 78 65  ....*....zxe.zxe
02801490   E4 7A 78 65  00 00 00 00  00 00 01 00  02 00 00 00  .zxe............
028014A0   00 00 00 00  01 00 00 00  01 04 00 00  00 00 00 00  ................
028014B0   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
028014C0   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
028014D0   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
028014E0   00 00 00 00  BC F3 45 A3  00 00 00 00  00 00 00 00  ......E.........
028014F0   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
```

## 31. Block δεδομένων.

### Tools:

Από την παραπάνω εντολή βλεπουμε `BLOCKS: 1025`.

### Hexedit:

Block: 01 04 00 00 (LE) -> 0x0401 = 1025

## 32. Μέγεθος αρχείου.

### Tools:

Από την παραπάνω εντολή βλεπουμε `Size: 42`. Άρα το μέγεθος είναι 42 Byt### Hexedit:es.

### Hexedit:

Size: 2A 00 00 00 (LE) -> 0x2a = 42 bytes.

## 33. Περιεχόμενα αρχείου.

### Tools:

Χρησιμοποιούμε την εντολή `debugfs -R "cat /dir2/helloworld" /dev/vdb`. Περιεχόμενο: Welcome to the Mighty World of Filesystems.

### Hexedit:

Θα δουμε το περιεχόμενο του data block με hexedit. offset = 1025 * 1024 = 1,049,600. Περιεχόμενο:

```
00100400   57 65 6C 63  6F 6D 65 20  74 6F 20 74  68 65 20 4D  Welcome to the M
00100410   69 67 68 74  79 20 57 6F  72 6C 64 20  6F 66 20 46  ighty World of F
00100420   69 6C 65 73  79 73 74 65  6D 73 00 00  00 00 00 00  ilesystems......
```
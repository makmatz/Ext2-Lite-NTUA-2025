## Εργαστήριο Λειτουργικών Συστημάτων | ΣΗΜΜΥ ΕΜΠ 2025-26
# Σύστήματα Αχείων, ext2-lite

#### Ομάδα oslab6
- **Μαντζώρος Γεράσιμος** (03122011)
- **Μωραΐτης Δημήτρης** (03122175)

----

# Μέρος 1ο

## Ερωτήσεις για fsdisk1.img

### 1. Προσθήκη `fdisk1.img` στο utopia.

Για να προσθέσουμε στην εικονική μηχανή utopia έναν επιπλέον δίσκο για την εικόνα `fdisk1.img`, προσθέτουμε την παράμετρο `- drive file=ext2-vdisks/fsdisk1.img,format=raw,if=virtio` στην εντολή εκκίνησης του QEMU μέσα στο αρχείο `utopia.sh`. Η συσκευή που προσθέσαμε εμφανίζεται στο utopia ως `/dev/vdb`.

### 2. Εύρεση μεγέθους δίσκου.

#### Tools:

Εκτελόυμε την εντολή `lsblk /dev/vdb`. Στο τερματικό εμφανίζεται `SIZE=50M` που σημαίνει ότι ο δίσκος έχει μέγεθος 50 MB.

#### Hexedit:

Εκτελώντας την εντολή `ls -lh fsdisk1.img` στον host βλέπουμε στο πεδίο για το μέγεθος του αρχείου την τιμή 50Μ.

### 3. Εύρεση τύπου ΣΑ.

#### Tools:

Εκτελόυμε την εντολή `lsblk -f /dev/vdb`. Στα μεταδεδομένα εμπεριέχεται `FSTYPE=ext2` που σημαίνει ότι το ΣΑ είναι ext2.

#### Hexedit:

Στο ext2, το superblock ξεκινά στο offset 1024 bytes. Το magic number βρίσκεται στο offset 0x38 του superblock. Εκτελούμε την εντολή `hexdump -s 1024 -n 64 fsdisk1.img`. Στα bytes 56 και 57 εμφανίζεται το magic number = 0xef53 που δηλώνει ότι το ΣΑ είναι τυπου ext.

### 4. Χρονοσφραγίδα δημιουργίας.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Filesystem created: Tue Dec 12 15:23:16 2023`. 

#### Hexedit:

Δεν περιλαμβάνεται.

### 5. Χρονοσφραγίδα τελευταίας προσάρτησης.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last mount time: Tue Dec 12 15:23:16 2023`.

#### Hexedit: 

Η χρονοσφραγίδα τελευταίας προσάρτησης βρίσκεται στο offset 0x2c του superblock. Εκτελούμε τις εντολές: `TS=$(hexdump -s $((1024+0x2C)) -n 4 -e '1/4 "%u"' /dev/vdb)`, `date -d "@$TS"`. Παίρνουμε `Tue Dec 12 03:23:16 PM UTC 2023`.

### 6. Μονοπάτι τελευταίας προσάρτησης.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last mounted on: /cslab-bunker`. 

#### Hexedit: 

Το μονοπάτι βρίσκεται σε offset 0x88. Εκτελούμε την εντολή `hexdump -s $((1024+0x88)) -n 64 -C /dev/vdb`. Αποτέλεσμα:
```
00000488  2f 63 73 6c 61 62 2d 62  75 6e 6b 65 72 00 00 00  |/cslab-bunker...|
00000498  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

### 7. Χρονοσφραγίδα τελευταίας τροποποίησης.

#### Tools: 

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Last write time: Tue Dec 12 15:23:17 2023`.

#### Hexedit: 

Η χρονοσφραγίδα τελευταίας τροποποίησης βρίσκεται στο offset 0x30 του superblock. Εκτελούμε τις εντολές: `TS=$(hexdump -s $((1024+0x30)) -n 4 -e '1/4 "%u"' /dev/vdb)`, `date -d "@$TS"`. Παίρνουμε `Tue Dec 12 03:23:17 PM UTC 2023`.

### 8. Ορισμός Block

Σε ένα σύστημα αρχείων το Block είναι η ελάχιστη μονάδα αποθήκευσης δεδομένων.

### 9. Block size

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Block size: 1024`. Άρα το Block size είναι 1 KB. 

#### Hexedit:

Σε offset 0x18 βρίσκουμε την τιμή s_log_block_size = 0, με την εντολή `LBS=$(hexdump -s $((1024+0x18)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $LBS`. BS = `1024 << s_log_block_size` = 1024.

### 10. Ορισμός Inode

Σε ένα σύστημα αρχείων το Inode είναι είναι μια δομή μεταδεδομένων που περιγράφει ένα αρχείο ή έναν κατάλογο. To Inode περιέχει τις εξής πληροφορίες:

- Τύπος αρχείου (character device, block device, regular file, directory κλπ).
- Δικαιώματα πρόσβασης.
- Αναγνωριστικό χρήστη / group.
- Χρονοσφραγίδες.
- Αριθμός Hard Links του αρχείου.
- Δείκτες στα Blocks δεδομένων του αρχείου.

### 11. Inode size.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχεται `Inode size: 128`.### Hexedit:

#### Hexedit:

Σε offset 0x58 βρίσκεται το inode size. Με την εντολή `hexdump -s $((1024+0x58)) -n 2 -e '1/2 "%u\n"' /dev/vdb` παίρνουμε 128.

### 12. Διαθέσιμα Blocks και Inodes.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Free blocks: 49552, Free inodes: 12810`.

#### Hexedit:

Το free blocks βρίσκεται σε offset 0x0c και το free inodes στο 0x10. Εκτελούμε τις εντολές `hexdump -s $((1024+0x0C)) -n 4 -e '1/4 "free_blocks=%u\n"' /dev/vdb` και `hexdump -s $((1024+0x10)) -n 4 -e '1/4 "free_inodes=%u\n"' /dev/vdb` και παίρνουμε free_blocks=49552 και free_inodes=12810.

### 13. Ορισμός Superblock.

Σε ένα σύστημα αρχείων το superblock είναι μια κεντρική δομή μεταδεδομένων που περιγράφει τη συνολική οργάνωση και κατάστασή του. Κάποιες από τις πληροφορίες που περιέχει είναι οι εξής:

- Τύπος συστήματος αρχείων.
- Block / Inode size.
- Συνολικός αριθμός Blocks / Inodes.
- Χρονοσφραγίδες.
- Πληροφορίες για τα groups του ΣΑ.

### 14. Θέση Superblock.

Στο σύστημα αρχείων ext2, το superblock βρίσκεται στο offset 1024 bytes (1 KB) από την αρχή του filesystem.

### 15. Εφεδρικά αντίγραφα Superblock.

Τα εφεδρικά αντίγραφα του superblock στο ext2 υπάρχουν για λόγους αξιοπιστίας και ανάκτησης, ώστε σε περίπτωση καταστροφής του κύριου superblock να μπορεί να αποκατασταθεί το σύστημα αρχείων και να διατηρηθεί η πρόσβαση στα δεδομένα.

### 16. Θέσεις αντιγράγων του Superblock.

#### Tools:

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

#### Hexedit:

Μπορούμε να υποθέσουμε ότι βρίσκονται στην αρχή κάθε group.

### 17. Block groups.

Ένα block group στο σύστημα αρχείων ext2 είναι μια λογική υποδιαίρεση του filesystem, που χωρίζει τον δίσκο σε μικρότερες, ενότητες για καλύτερη οργάνωση και αξιοπιστία. Κάθε Block group, εκτός από τα Data Blocks, περιέχει και άλλες χρήσιμες πληροφορίες όπως:

- Αντίγραφο Superblock.
- Block bitmap.
- Inode bitmap.
- Inode table.
- Block group descriptor.

### 18. Πλήθος Block groups.

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

### 19. Αριθμός Blocks groups στο `fdisk1.img`.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Block count: 51200, Blocks per group: 8192`. Από τον παραπάνω τύπο προκύπτει ότι έχουμε 7 groups.

#### Hexedit:

Ο αριθμός των blocks βρίσκεται σε offset 0x04 και των blocks per group στο 0x20. Χρησιμοποιούμε τις εντολές `TOTAL=$(hexdump -s $((1024+0x04)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $TOTAL`, `BPG=$(hexdump -s $((1024+0x20)) -n 4 -e '1/4 "%u"' /dev/vdb); echo $BPG` και `echo $(( (TOTAL + BPG - 1) / BPG ))` και παίρνουμε το αποτέλεσμα: 7.

### 20. Block group descriptors.

Ο block group descriptor στο ext2 είναι μια δομή μεταδεδομένων που περιγράφει τη διάταξη και την κατάσταση ενός block group, αποθηκεύοντας πληροφορίες για τη θέση των bitmaps, του inode table και τον αριθμό των ελεύθερων πόρων.

### 21. Εφεδρικά αντίγραφα Block group descriptors.

Τα εφεδρικά αντίγραφα των block group descriptors στο σύστημα αρχείων ext2 υπάρχουν για λόγους αξιοπιστίας και ανάκτησης, καθώς οι descriptors περιγράφουν τη θέση και την κατάσταση των βασικών δομών κάθε block group. Σε περίπτωση καταστροφής του κύριου πίνακα descriptors, τα εφεδρικά αντίγραφα επιτρέπουν την αποκατάσταση του filesystem και τη διατήρηση της πρόσβασης στα δεδομένα.

### 22. Θέση bgd αντίγραφων.

#### Tools:

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

#### Hexedit:

Γνωρίζουμε ότι βρίσκονται μετά από τα εφεδρικά αντίγραφα των superblocks.

### 23. Block / Inode bitmap.

Το block bitmap και το inode bitmap στο ext2 είναι δομές μεταδεδομένων που αποθηκεύουν σε μορφή bit την κατάσταση (ελεύθερο ή δεσμευμένο) των blocks και των inodes αντίστοιχα και βρίσκονται μέσα σε κάθε block group, με τις ακριβείς θέσεις τους να καθορίζονται από τους block group descriptors.

### 24. Inode tables.

Τα inode tables στο ext2 είναι περιοχές του δίσκου που αποθηκεύουν τις δομές inode και βρίσκονται μέσα σε κάθε block group, μετά τα bitmaps, με τη θέση τους να καθορίζεται από τους block group descriptors.

### 25. Inodes.

Κάθε inode στο ext2 περιέχει μεταδεδομένα ενός αρχείου, όπως δικαιώματα, μέγεθος, χρονικές σφραγίδες και δείκτες προς data blocks, και αποθηκεύεται μέσα στα inode tables που βρίσκονται σε κάθε block group του filesystem.

### 26. Αριθμός Blocks / Inodes.

#### Tools:

Εκτελούμε την εντολή `tune2fs - l /dev/vdb`. Στα δεδομένα του superblock του ΣΑ εμπεριέχονται `Blocks per group: 8192, Inodes per group: 1832`.

#### Hexedit:

To blocks per group βρίσκεται σε offset 0x20 και το inodes per group στο 0x28. Με τις εντολές `hexdump -s $((1024+0x20)) -n 4 -e '1/4 "blocks_per_group=%u\n"' /dev/vdb`, `hexdump -s $((1024+0x28)) -n 4 -e '1/4 "inodes_per_group=%u\n"' /dev/vdb` παίρνουμε `blocks_per_group=8192, inodes_per_group=1832`.

### 27. Εύρεση inode του `/dir/helloworld`.

#### Tools:

Εκτελούμε την εντολη `debugfs -R "stat /dir2/helloworld" /dev/vdb`. Στην πρώτη γραμμή εμφανίζεται το inode που αντιστοιχεί το συγκεκριμένο αρχείο, `Inode: 9162`.

#### Hexedit:

Με hexedit στο αρχείο `/dev/vdb` ψάχνουμε το string "helloworld" (68656c6c6f776f726c64). Υπάρχει μία μόνο φορά, άρα η τιμή 8 bytes πριν από αυτο αντιστοιχεί στο αντίστοιχο inode. Η τιμή αυτή είναι 0xca23 (LE) = 9162.

### 28. Εύρεση group του `/dir/helloworld`.

Τα inodes κατανέμονται σειριακά στα groups και έχουμε 1832 inodes per group. Αρα το 9162 βρίσκεται στο group 5.

### 29. Εύρεση Inode table του Inode 9162.

#### Tools:

Το συγκεκριμένο Inode βρίσκεται, όπως είδαμε, στο group 5. Εκτελούμε την εντολή `dumpe2fs /dev/vdb | grep -A 5 "Group 5"`. Βλέπουμε `Inode table at 40965-41193`.

#### Hexedit:

Στο Block 2 υπάρχει το group descriptor table. Σε offset 2048+(5×32)=2048+160=2208=0x8a0 βρίσκεται το group descriptor του group 6. Τα bytes 8 - 11 δείχνουν το block που ξεκινάει το inode table. Με hexedit βρίσκουμε 05 A0 (LE) -> 0xa005 = 40965.

### 30. Περιεχόμενα του Inode 9162.

#### Tools:

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

#### Hexedit:

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

### 31. Block δεδομένων.

#### Tools:

Από την παραπάνω εντολή βλεπουμε `BLOCKS: 1025`.

#### Hexedit:

Block: 01 04 00 00 (LE) -> 0x0401 = 1025

### 32. Μέγεθος αρχείου.

#### Tools:

Από την παραπάνω εντολή βλεπουμε `Size: 42`. Άρα το μέγεθος είναι 42 Byt### Hexedit:es.

#### Hexedit:

Size: 2A 00 00 00 (LE) -> 0x2a = 42 bytes.

### 33. Περιεχόμενα αρχείου.

#### Tools:

Χρησιμοποιούμε την εντολή `debugfs -R "cat /dir2/helloworld" /dev/vdb`. Περιεχόμενο: Welcome to the Mighty World of Filesystems.

#### Hexedit:

Θα δουμε το περιεχόμενο του data block με hexedit. offset = 1025 * 1024 = 1,049,600. Περιεχόμενο:

```
00100400   57 65 6C 63  6F 6D 65 20  74 6F 20 74  68 65 20 4D  Welcome to the M
00100410   69 67 68 74  79 20 57 6F  72 6C 64 20  6F 66 20 46  ighty World of F
00100420   69 6C 65 73  79 73 74 65  6D 73 00 00  00 00 00 00  ilesystems......
```

----

## Ερωτήσεις για fsdisk2.img

### 1. Προσθήκη εοκόνας `fdisk2`.

Για να προσθέσουμε στην εικονική μηχανή utopia έναν επιπλέον δίσκο για την εικόνα `fdisk2.img`, προσθέτουμε την παράμετρο `- drive file=ext2-vdisks/fsdisk2.img,format=raw,if=virtio` στην εντολή εκκίνησης του QEMU μέσα στο αρχείο `utopia.sh`. Η συσκευή που προσθέσαμε εμφανίζεται στο utopia ως `/dev/vdb`. Για να προσαρτήσουμε την εικόνα στον κατάλογο `/mnt` χρησιμοποιούμε την εντολή `mount /dev/vdb /mnt`.

### 2. Δημιουργία αρχείου στο ΣΑ.

Για να δημιουργήσουμε ένα νέο αρχείο με όνομα `file1` στο ΣΑ της εικόνας χρησιμοποιούμε την εντολή `touch /mnt/file1`.

### 3. Αποτυχία εντολής.

Η παραπάνω αντολή απέτυχε επειδή δεν υπάρχει διαθέσιμος χώρος στο ΣΑ.

### 4. Κλήση συστήματος της `touch`.

Με χρήση `strace` παρατηρούμε ότι η εντολή `touch` εκτελέι την κλήση συστήματος `openat(AT_FDCWD, "file1", O_WRONLY | O_CREAT | O_NOCTTY | O_NONBLOCK, 0666)`. Η εντολή αποτυχγάνει με κωδικό λάθους ENOSPC (-1): No space left on device.

### 5. Πλήθος αρχείων και καταλόγων.

#### Tools:

Χρησιμοποιούμε την εντολή `df -i /mnt`. Αυτή επιστρπεφει IUsed = 5136, που σημαίνει ότι το ΣΑ περιέχει 5136 αρχεία και καταλόγους.

#### Hexedit:

Χρησιμοποιούμε hexedit και βλέπουμε τα 4 bytes σε offset 0x400 (αρχή superblock). Το περιεχόμενό τους είναι 10 14 00 00 = 0x1014 (little endian), άρα inodes_count = 5135. Βλέπουμε τα 4 bytes σε offset 0x410. Το περιεχόμενο είναι 00 00 00 00, άρα free_inodes = 0. Άρα το πλήθος των αρχείων και καταλόγων είναι 5135. 

### 6. Χρησιμοποιούμενος χώρος ΣΑ.

#### Tools:

Χρησιμοποιούμε την εντολή `df -h /mnt`. Αυτή επιστρέφει Used: 270K. Άρα τα δεδομένα καταλαμβάνουν συνολικά 270K bytes. Με την εντολή `tune2fs -l /dev/vdb` βρίσκουμε Overhead clusters: 655. Άρα τα μεταδεδομένα έχουν μέγεθος 655 * block_size = 655K. Συνολικά ο χρησιμοποιούμενος χώρος είναι 270Κ + 655Κ = 925Κ.

#### Hexedit:

Offset 0x418: 00 -> block_size = 1024 * 2^0 bytes = 1K.

Offset 0x404: 00 50 00 00 -> blocks_count = 0x5000 = 20480.

Offset 0x40C: 63 4C 00 00 -> free_blocks = 0x4C63 = 19555.

Used space = used_blocks * block size = 925K

### 7. Μέγεθος ΣΑ.

### Tools:

Χρησιμοποιούμε την εντολή `df -h /mnt`. Αυτή επιστρέφει Size: 20M. Άρα το ΣΑ έχει μέγεθος 20M bytes.

#### Hexedit:

Size = total_blocks * block_size = 5 * 16^3 K = 5 * 2^12 K = 20M

### 8. Ελεύθερος χώρος ΣΑ.

#### Tools:

Εκτελούμε την εντολή `df /mnt`. Αυτή επιστρέφει Available: 19555. Άρα υπάρχει ελεύθερος χώρος στο ΣΑ (19555 blocks).

#### Hexedit:

Υπολογίσαμε ότι free_blocks = 19555, άρα υπάρχει ελεύθερος χώρος.

### 9. Αιτία αφάλματος.

Ο λόγος που δεν μπορούμε να δημιουργήσουμε νέο αρχείο στο ΣΑ παρότι υπάρχει ελεύθερος χώρος είναι ότι δεν υπάρχουν διαθέσιμα inodes.

----

## Ερωτήσεις για fsdisk3.img

### 1. Εργαλείο ελέγχου ΣΑ.

Το εργαλείο στο Linux για τον έλεγχο και την επιδιόρθωση συστημάτων αρχείων της οικογένειας ext είναι το e2fsck.

### 2. Αιτίες και τύποι αλλοιώσεων.

Οι αλλοιώσεις (corruption) προκύπτουν συνήθως από απότομη διακοπή ρεύματος, αστοχία υλικού (bad sectors), ή σφάλματα στον kernel/drivers.

Δέκα ενδεικτικές αλλοιώσεις:

- Incorrect Superblock Checksum: Ο έλεγχος ακεραιότητας του superblock αποτυγχάνει.

- Free Blocks Count Mismath: Η τιμή των ελεύθερων blocks στο superblock δεν συμφωνεί με το bitmap.

- Inode Bitmap Mismatch: Ένα inode σημειώνεται ως χρησιμοποιημένο ενώ είναι ελεύθερο (ή το αντίστροφο).

- Zero Dtime on Deleted Inode: Ένα διαγραμμένο inode δεν έχει σωστή χρονοσήμανση διαγραφής.

- Invalid Inode Mode: Το inode περιέχει μη έγκυρα δικαιώματα ή τύπο αρχείου.

- Directory Entry Link Count: Ο αριθμός των συνδέσμων (hard links) προς έναν κατάλογο είναι λάθος.

- Overlapping Blocks: Δύο διαφορετικά inodes ισχυρίζονται ότι κατέχουν το ίδιο block δεδομένων.

- Invalid Inode Size: Το μέγεθος που αναφέρεται στο inode δεν αντιστοιχεί στα blocks που καταλαμβάνει.

- Bad Directory Entry: Το όνομα ενός αρχείου περιέχει μη έγκυρους χαρακτήρες ή το μήκος του entry είναι λάθος.

- Unconnected Inode: Ένα inode που περιέχει δεδομένα αλλά δεν συνδέεται με κανέναν κατάλογο (ορφανό).

### 3. Επιδιόρθωση με fsck.

Εκτελούμε την εντολή `fsck.ext2 -y fdisk3.img`.

Αποτέλεσμα:
```
fsdisk3.img contains a file system with errors, check forced.
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
First entry 'BOO' (inode=1717) in directory inode 1717 (/dir-2) should be '.'
Fix? yes

Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Inode 3425 ref count is 1, should be 2.  Fix? yes

Pass 5: Checking group summary information
Block bitmap differences:  +34
Fix? yes

Free blocks count wrong (926431538, counted=19800).
Fix? yes
```

### 4 & 5. Εντοπισμος και διόρθωση με hexedit.

1. Στο directory που αντιστοιχεί στο inode 1717 η πρώτη καταχώρηση έχει όνομα 'BOO', ενώ θα έπρεπε να έχει όνομα '.'. Με hexedit ψάχνουμε το string 'BOO' στο αρχείο. Αυτό βρίσκεται στη θέση 0x00837008. Το αλλάζουμε σε '.\0\0'. Εκτελώντας πάλι τον έλεγχο παρατηρούμε ότι αυτό το σφάλμα δεν εμφανίζεται πια.

2. Στο inode 3425 το reference count είναι 1, ενώ θα έπρεπε να είναι 2. Έχουμε 1712 inodes per group άρα το inode 3425 είναι το πρώτο του group 2. To inode αυτό βρίσκεται στη διεύθυνση 0x1001400. Η τιμή i_links_count βρίσκεται σε offset 0x1A μέσα στο inode, άρα στη διεύθυνση 0x100141A. Με hexedit πηγαίνουμε σε αυτή τη διεύθυνση και αλλάζουμε την τιμή από 01 σε 02. Εκτελώντας πάλι τον έλεγχο παρατηρούμε ότι αυτό το σφάλμα δεν εμφανίζεται πια.

3. Το block 34 πρέπει να σημειωθεί ως χρησιμοποιημένο. Το block bitmap ξεκινά στο block 0xC00. Το block 34 αντιστοιχεί στο bit 6 (little endian) του byte στη διεύθυνση 0xC04. To byte αυτό έχει τιμή FD = 1111 1101 και πρέπει να πάρχει την τιμή FF = 1111 1111. Εκτελώντας πάλι τον έλεγχο παρατηρούμε ότι αυτό το σφάλμα δεν εμφανίζεται πια.

4. To free_blocks_count είναι 926431538, ενώ θα έπρεπε να είναι 19800. Η τιμή αυτη βρίσκεται στο superblock στη διεύθυνση 0x40C. Αντικαθιστούμε τα 4 bytes σε αυτό το offset (και κάθε +0xC για τα αντίγραφα του superblock) με 58 4D 00 00. Εκτελώντας πάλι τον έλεγχο παρατηρούμε ότι αυτό το σφάλμα δεν εμφανίζεται πια.

----

# Μέρος 2ο

Για το δεύτερο μέρος της άσκησης καλούμαστε να ολοκληρώσουμε την υλοποίηση ενός kernel module για το σύστημα αρχείων ext2-lite. Οι συναρτήσεις που υλοποιήσαμε είναι οι εξής:

### 1. `init_ext2_fs` και `exit_ext2_fs` (super.c)

* Η συνάρτηση `init_ext2_fs` καλεί την `init_inodecache()` για να αρχικοποιήσει την cache των inodes και στη συνέχεια καταχωρεί το σύστημα αρχείων "ext2-lite" στον πυρήνα του Linux καλώντας την `register_filesystem(&ext2_fs_type)`.
* Αντίστοιχα, η `exit_ext2_fs` πραγματοποιεί την αντίστροφη διαδικασία, διαγράφοντας το σύστημα αρχείων με την `unregister_filesystem(&ext2_fs_type)` και αποδεσμεύοντας τη μνήμη της cache με την `destroy_inodecache()`.

```c
static int __init init_ext2_fs(void)
{
	ext2_debug("Initialize inode cache");
	int err = init_inodecache();
	if (err)
		return err;

	ext2_debug("Register filesystem");
	err = register_filesystem(&ext2_fs_type);
	if (err) {
		ext2_debug("Failed to register filesystem");
		destroy_inodecache();
	}

	return err;
}

static void __exit exit_ext2_fs(void)
{
	unregister_filesystem(&ext2_fs_type);
	destroy_inodecache();
}
```

### 2. `ext2_find_entry` (dir.c)

* Η συνάρτηση σαρώνει τις σελίδες ενός καταλόγου (καλώντας την `ext2_get_folio`) για να βρει ένα αρχείο με συγκεκριμένο όνομα.
* Χρησιμοποιεί την `ext2_last_byte` για να βρει το όριο αναζήτησης σε κάθε σελίδα και διατρέχει τα εγγραφές με την `ext2_next_entry`.
* Αν η `ext2_match` επιβεβαιώσει ταύτιση ονόματος, επιστρέφει την εγγραφή, αλλιώς αποδεσμεύει τη σελίδα καλώντας την `folio_release_kmap`.

```c
ext2_dirent *ext2_find_entry(struct inode *dir, const struct qstr *child,
                             struct folio **foliop)
{
	const char *name = child->name;
	int namelen = child->len;
	unsigned reclen = EXT2_DIR_REC_LEN(namelen);
	unsigned long npages = dir_pages(dir);
	unsigned long i;
	ext2_dirent *de;
	char *kaddr;
	struct folio *folio;
	char *limit;

	if (npages == 0)
		return ERR_PTR(-ENOENT);

	/* Scan all the pages of the directory to find the requested name. */
	for (i=0; i < npages; i++) {
		kaddr = ext2_get_folio(dir, i, 0, &folio);
		if (IS_ERR(kaddr))
			return ERR_CAST(kaddr);

		limit = kaddr + ext2_last_byte(dir, i) - EXT2_DIR_REC_LEN(1);
		de = (ext2_dirent *)kaddr;
		for ( ; (char *)de <= limit; de = ext2_next_entry(de)) {
			if (de->rec_len == 0) {
				ext2_error(dir->i_sb, __func__, "zero-length directory entry");
				folio_release_kmap(folio, kaddr);
				return ERR_PTR(-EIO);
			}
			if (ext2_match(namelen, name, de)) {
				*foliop = folio;
				return de;
			}
		}
		folio_release_kmap(folio, kaddr);
	}
	return ERR_PTR(-ENOENT);
}
```

### 3. `ext2_get_inode` (inode.c)

* Εντοπίζει ένα συγκεκριμένο inode στον δίσκο. Αρχικά, υπολογίζει σε ποιο block group ανήκει το inode και ανακτά τον αντίστοιχο group descriptor.
* Στη συνέχεια, υπολογίζει το ακριβές block του inode table και το byte offset μέσα σε αυτό.
* Τέλος, διαβάζει το block στη μνήμη χρησιμοποιώντας την `sb_bread` και επιστρέφει έναν δείκτη (`struct ext2_inode *`) στα δεδομένα του.

```c
static struct ext2_inode *ext2_get_inode(struct super_block *sb, ino_t ino,
                                         struct buffer_head **p)
{
	struct buffer_head *bh;
	unsigned long block_group;
	unsigned long block;
	unsigned long offset;
	struct ext2_group_desc *gdp;
	unsigned long inodes_pg = EXT2_INODES_PER_GROUP(sb);
	int inode_sz = EXT2_INODE_SIZE(sb);
	unsigned long blocksize = sb->s_blocksize;

	*p = NULL;
	/* Check the validity of the given inode number. */
	if ((ino != EXT2_ROOT_INO && ino < EXT2_FIRST_INO(sb)) ||
	    ino > le32_to_cpu(EXT2_SB(sb)->s_es->s_inodes_count))
		goto einval;

	/* Figure out in which block is the inode we are looking for and get
	 * its group block descriptor. */
	block_group = (ino - 1) / inodes_pg;
	gdp = ext2_get_group_desc(sb, block_group, NULL);
	if (!gdp)
		return ERR_PTR(-EIO);

	/* Figure out the offset within the block group inode table */
	offset = (ino - 1) % inodes_pg;
	block = le32_to_cpu(gdp->bg_inode_table) +
		(offset / EXT2_SB(sb)->s_inodes_per_block);
	offset = (offset % EXT2_SB(sb)->s_inodes_per_block) * inode_sz;

	/* Return the pointer to the appropriate ext2_inode */
	bh = sb_bread(sb, block);
	if (!bh)
		goto eio;

	ext2_debug("Assigning block");
	*p = bh;
	return (struct ext2_inode *)(bh->b_data + offset);
einval:
	ext2_error(sb, __func__, "bad inode number: %lu", (unsigned long)ino);
	return ERR_PTR(-EINVAL);
eio:
	ext2_error(sb, __func__, "unable to read inode block - inode=%lu, block=%lu",
	           (unsigned long)ino, block);
	return ERR_PTR(-EIO);
}
```

### 4. `ext2_iget` (inode.c)

* Δημιουργεί (ή ανακτά) ένα VFS inode στη μνήμη, διαβάζοντας πρώτα τα ακατέργαστα (raw) δεδομένα του από τον δίσκο μέσω της `ext2_get_inode`.
* Έπειτα, ανάλογα με τον τύπο του αρχείου (π.χ. κανονικό αρχείο με `S_ISREG`, κατάλογος με `S_ISDIR`), αρχικοποιεί τα πεδία `inode->i_op` (inode operations), `inode->i_fop` (file operations) και `inode->i_mapping->a_ops` (address space operations) με τις αντίστοιχες συναρτήσεις του ext2-lite.
* Η αρχική υλοποίηση είχε την γραμμή `ei->i_dtime = le32_to_cpu(raw_inode->i_dtime);` προτού αρχικοποιηθεί η μεταβλητή `ei` οπότε χρειάστηκε να μεταφέρουμε την γραμμή αυτή αργότερα στον κώδικα για να αποφύγουμε null dereference.

```c
struct inode *ext2_iget(struct super_block *sb, unsigned long ino)
{
	struct ext2_inode_info *ei;
	struct buffer_head *bh = NULL;
	struct ext2_inode *raw_inode;
	struct inode *inode;
	long ret = -EIO;
	int n;

	ext2_debug("request to get ino: %lu\n", ino);

	/*
	 * Allocate the VFS node.
	 * We know that the returned inode is part of a bigger ext2_inode_info
	 * inode since iget_locked() calls our ext2_sops->alloc_inode() function
	 * to perform the allocation of the inode.
	 */
	inode = iget_locked(sb, ino);
	if (!inode)
		return ERR_PTR(-ENOMEM);
	if (!(inode->i_state & I_NEW))
		return inode;

	/*
	 * Read the EXT2 inode *from disk*
	 */
	raw_inode = ext2_get_inode(inode->i_sb, ino, &bh);
	if (IS_ERR(raw_inode)) {
		ret = PTR_ERR(raw_inode);
		brelse(bh);
		iget_failed(inode);
		return ERR_PTR(ret);
	}

	/*
	 * Fill the necessary fields of the VFS inode structure.
	 */
	inode->i_mode = le16_to_cpu(raw_inode->i_mode);
	i_uid_write(inode, (uid_t)le16_to_cpu(raw_inode->i_uid));
	i_gid_write(inode, (gid_t)le16_to_cpu(raw_inode->i_gid));
	set_nlink(inode, le16_to_cpu(raw_inode->i_links_count));
	inode_set_atime(inode, (signed)le32_to_cpu(raw_inode->i_atime), 0);
	inode_set_ctime(inode, (signed)le32_to_cpu(raw_inode->i_ctime), 0);
	inode_set_mtime(inode, (signed)le32_to_cpu(raw_inode->i_mtime), 0);
	inode->i_blocks = le32_to_cpu(raw_inode->i_blocks);
	inode->i_size = le32_to_cpu(raw_inode->i_size);
	if (i_size_read(inode) < 0) {
		ret = -EUCLEAN;
		brelse(bh);
		iget_failed(inode);
		return ERR_PTR(ret);
	}

	//> Setup the {inode,file}_operations structures depending on the type.
	if (S_ISREG(inode->i_mode)) {
		inode->i_op = &ext2_file_inode_operations;
		inode->i_fop = &ext2_file_operations;
		inode->i_mapping->a_ops = &ext2_aops;
	} else if (S_ISDIR(inode->i_mode)) {
		inode->i_op = &ext2_dir_inode_operations;
		inode->i_fop = &ext2_dir_operations;
		inode->i_mapping->a_ops = &ext2_aops;
	} else if (S_ISLNK(inode->i_mode)) {
		if (ext2_inode_is_fast_symlink(inode)) {
			inode->i_op = &simple_symlink_inode_operations;
			inode->i_link = (char *)ei->i_data;
			nd_terminate_link(ei->i_data, inode->i_size, sizeof(ei->i_data) - 1);
		} else {
			inode->i_op = &page_symlink_inode_operations;
			inode_nohighmem(inode);
			inode->i_mapping->a_ops = &ext2_aops;
		}
	} else {
		inode->i_op = &ext2_special_inode_operations;
		if (raw_inode->i_block[0])
			init_special_inode(inode, inode->i_mode,
			   old_decode_dev(le32_to_cpu(raw_inode->i_block[0])));
		else 
			init_special_inode(inode, inode->i_mode,
			   new_decode_dev(le32_to_cpu(raw_inode->i_block[1])));
	}

	/*
	 * Fill the necessary fields of the ext2_inode_info structure.
	 */
	ei = EXT2_I(inode);
	ei->i_dtime = le32_to_cpu(raw_inode->i_dtime);
	ei->i_flags = le32_to_cpu(raw_inode->i_flags);
	ext2_set_inode_flags(inode);
	ei->i_dtime = 0;
	ei->i_state = 0;
	ei->i_block_group = (ino - 1) / EXT2_INODES_PER_GROUP(inode->i_sb);
	//> NOTE! The in-memory inode i_data array is in little-endian order
	//> even on big-endian machines: we do NOT byteswap the block numbers!
	for (n = 0; n < EXT2_N_BLOCKS; n++)
		ei->i_data[n] = raw_inode->i_block[n];

	brelse(bh);
	unlock_new_inode(inode);
	return inode;
}
```

### 5. `ext2_allocate_in_bg` (balloc.c)

* Εντοπίζει το πρώτο διαθέσιμο bit (ελεύθερο block) στο block bitmap του δοσμένου group χρησιμοποιώντας την `find_next_zero_bit_le`.
* Αφού το μαρκάρει ως δεσμευμένο με την `ext2_set_bit_atomic`, επιχειρεί μέσα σε ένα βρόχο να δεσμεύσει επιπλέον συνεχόμενα blocks έως ότου φτάσει το όριο του ορίσματος `count` ή συναντήσει δεσμευμένο block.
* Τέλος, ενημερώνει την παράμετρο `count` με το πλήθος των blocks που όντως δεσμεύτηκαν και επιστρέφει το offset του πρώτου block.

```c
static int ext2_allocate_in_bg(struct super_block *sb, int group,
                               struct buffer_head *bitmap_bh, unsigned long *count)
{
	ext2_fsblk_t group_first_block = ext2_group_first_block_no(sb, group);
	ext2_fsblk_t group_last_block = ext2_group_last_block_no(sb, group);
	ext2_grpblk_t nblocks = group_last_block - group_first_block + 1;
	ext2_grpblk_t first_free_bit;
	unsigned long num;
	struct ext2_sb_info *sbi = EXT2_SB(sb);

	first_free_bit = 0;
	while (1) {
		first_free_bit = find_next_zero_bit_le(bitmap_bh->b_data, nblocks, first_free_bit);
		if (first_free_bit >= nblocks)
			return -1;
		if (!ext2_set_bit_atomic(sb_bgl_lock(sbi, group), first_free_bit, bitmap_bh->b_data))
			break;
		first_free_bit++;
	}

	for (num = 1; num < *count; num++) {
		if (first_free_bit + num >= nblocks)
			break;
		if (ext2_set_bit_atomic(sb_bgl_lock(sbi, group), first_free_bit + num, bitmap_bh->b_data))
			break;
	}

	*count = num;
	return first_free_bit;
}
```


----

# Πηγές - Βιβλιογραφία


- Εκφώνηση άσκησης (https://helios.ntua.gr/mod/resource/view.php?id=5433)
- Οδηγός εργαστηριακής άσκησης, Συστήματα Αρχείων στο Linux (https://helios.ntua.gr/mod/resource/view.php?id=41456)
- Linux Kernel documentation (https://docs.kernel.org/)

----

# Άδεια

©2025 Σιακαβάρας Δ., Μωραΐτης Δ., Μαντζώρος Γ.

GNU General Public License v2

Σύνδεσμος GitLab: https://gitlab.com/xheonin/ext2-lite-oslab6

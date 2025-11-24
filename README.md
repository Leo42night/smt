# Tugas Besar NLP
Mata Kuliah: Pemrosesan Bahasa Alami (2025 Ganjil).
- Link Dataset [Kaggle.com / Akhmad Fakhri / Indo - Melayu Sambas](https://www.kaggle.com/datasets/ahmdfkhr3/translasi-bahasa-indonesia-melayu-sambas)
- [Link Laporan](https://docs.google.com/document/d/13kArKcZxSMtIK1NfY1xjrmnPuMkNDk_aJipFPJufNJQ/edit?usp=sharing)

## Setup
- Tutorial ini menggunakan terminal WSL Di Debian trixie (Debian 12/13), jadi mungkin akan ada perbedaan dependensi.
- Donwload [Code Moses versi 4 (2017) dan Binary nya](http://www2.statmt.org/moses/?n=moses.releases), sesuaikan binary dengan versi repo anda.
- Download [tools tambahan Moses](https://drive.google.com/drive/folders/1A71fw5uArOPfU4OaS6eGtJrvVVll61r4?usp=drive_link) (mkcls, GIZA++, & snt2cooc.out), file ini perlu ada di sub-folder `bin/`.
- [Srilm](https://drive.google.com/file/d/1xaxYrukhW1BtusUVeUia3TWu2suDTs-9/view?usp=drive_link)

## Moses
Apa itu Moses?<br>
Moses adalah open-source Statistical Machine Translation (SMT) framework.
Ini digunakan untuk membangun mesin penerjemah berbasis phrase-based SMT atau hierarchical SMT.

Sederhananya:
> Moses = alat untuk melatih model penerjemahan otomatis berbasis statistik
> (bukan neural seperti Transformer / NMT modern).

## Config Path
`$ROOTPATH` Adalah contoh direktori penghubung proyek dan utilitas. Anda bebas meletakan **Moses** dan **Slilm** dimanapun.
```bash
export pathsmt="$ROOTPATH/smt" # root project ini
export pathmoses="$ROOTPATH/moses"
export pathsrilm="$ROOTPATH/srilm"
```
| Variabel      | Isi                        | Fungsi                                    |
| ------------- | -------------------------- | ----------------------------------------- |
| **pathsmt**   | Folder proyek SMT kamu     | tempat penyimpanan korpus/pipeline        |
| **pathmoses** | Instalasi Moses            | berisi script training/tokenizing/decoder |
| **pathsrilm** | Instalasi SRILM            | untuk membuat language model              |

## Proses
Id: Indonesia, Smb: Sambas.

```bash
cd $pathsmt
# 1. Membersihkan korpus
$pathmoses/scripts/training/clean-corpus-n.perl corpus/korpus id smb corpus/clean 1 20
# output: corpus/ > clean.id & clean.smb
# 2. Lowercasing
$pathmoses/scripts/tokenizer/lowercase.perl < corpus/clean.id > corpus/lowercased.id
$pathmoses/scripts/tokenizer/lowercase.perl < corpus/clean.smb > corpus/lowercased.smb
# output: corpus > lowercased.id & smb

# 3. Tokenization
$pathmoses/scripts/tokenizer/tokenizer.perl < corpus/lowercased.id > corpus/tokenized.id
$pathmoses/scripts/tokenizer/tokenizer.perl < corpus/lowercased.smb > corpus/tokenized.smb
# output: tokenized id & smb
```

## Training Language Model
```bash
cd $pathsmt
mkdir lm
$pathsrilm/bin/i686-m64/ngram-count -order 3 interpolate -unk -text corpus/tokenized.id -lm lm/id.lm
```

## Tranining Translation Model
```bash
cd $pathsmt
$pathmoses/scripts/training/train-model.perl -root-dir . --corpus corpus/tokenized --f smb --e id --lm 0:3:$pathsmt/lm/id.lm -external-bin-dir $pathmoses/bin
```

## Test Mesin
```bash
cd $pathsmt
$pathmoses/bin/moses -f model/moses.ini
```
Jika dapat output `Created input-output object : [0.xxx] seconds` Berarti Model sudah siap  digunakan. Silakan test input file berikut.

## Uji Akurasi
```bash
cd $pathsmt
cp corpus/tokenized.id ref.txt
cp corpus/tokenized.smb intest.txt
$pathmoses/bin/moses -f model/moses.ini < intest.txt > out.txt
$pathmoses/scripts/generic/multi-bleu.perl ref.txt < out.txt
```

Hasil Akhir sebagai berikut:
```bash
.
├── corpus
│   ├── clean.id
│   ├── clean.smb
│   ├── id-smb-int-train.snt
│   ├── id.vcb
│   ├── id.vcb.classes
│   ├── id.vcb.classes.cats
│   ├── korpus.id
│   ├── korpus.smb
│   ├── lowercased.id
│   ├── lowercased.smb
│   ├── smb-id-int-train.snt
│   ├── smb.vcb
│   ├── smb.vcb.classes
│   ├── smb.vcb.classes.cats
│   ├── tokenized.id
│   └── tokenized.smb
├── giza.id-smb
│   ├── id-smb.A3.final.gz
│   ├── id-smb.cooc
│   └── id-smb.gizacfg
├── giza.smb-id
│   ├── smb-id.A3.final.gz
│   ├── smb-id.cooc
│   └── smb-id.gizacfg
├── indo-sambas.csv
├── intest.txt
├── lm
│   └── id.lm
├── log.txt
├── model
│   ├── aligned.grow-diag-final
│   ├── extract.inv.sorted.gz
│   ├── extract.sorted.gz
│   ├── lex.e2f
│   ├── lex.f2e
│   ├── moses.ini
│   └── phrase-table.gz
├── out.txt
├── README.md
├── ref.txt
└── split-corpus.ipynb

6 directories, 37 files
```

## Tips & Tutorial
- Cari path program: `which <file_execute>`
- Cari by name: `find / -name moses 2>/dev/null`
- Buat env variabel:
```bash
$ nano ~/.bashrc
# isi file ini: export ROOTPATH="/input/your/path" # root project ini
# Ctrl+X untuk keluar, Y untuk save.
$ source ~/.bashrc
# cara cek: echo $ROOTPATH
```
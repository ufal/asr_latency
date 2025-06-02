# ASR Latency with Continuous Levenshtein Alignment

This is a script to calculate the latency of streaming ASR (aka low-latency, incremental, simultaneous, or "real-time" ASR) that emits the partial outputs gradually, simultaneously with receiving the partial input as it is being uttered by humans.

This is a novel more robust and reliable approach featuring:

- counting with word-level latency. It requires gold word-level timestamps, which can be generated with a forced aligner from any audio-gold transcript pair.
- computing more accurate character-level alignment of the gold and ASR candidate, which is more robust to subtle deviations in e.g. endings than word-level alignment
- avoiding coincident alignment to inserted or deleted word, which would wrongly align too late or too early

The disadvantage is the quadratic computation complexity, but it is still feasible. 11 minutes minutes of transcripts can be computed in 1 minute. Long transcripts can be segmented.

**Reference:** Section 5.1 in the paper "Simultaneous Translation with Offline Speech and LLM Models in CUNI Submission to IWSLT 2025", Macháček and Polák, 2025

## Usage

1. **Installation:** The script `latency.py` has no dependencies, only python3 is needed.

2. **Files format:**

- gold transcript must be a file with this tab-separated columns:

```
0.753	1.113	Hello,
1.2429999999999999	1.443	this
1.443	1.593	is
1.593	1.833	Jiawei
1.833	2.193	Zhou
2.193	2.443	from
2.443	2.7430000000000003	Harvard
2.7430000000000003	3.423	University.
3.914	3.9939999999999998	I
3.9939999999999998	4.134	am
```
 - column1, column2: beginning and end time where this word is uttered in the audio, in seconds
 - column3: the word

- the ASR candidate format:

```
2600.0000 764 2600  Hello, this is
4440.0000 2600 4440  Jiawei Zhou from Harvard
6280.0000 4440 6280  University. I am very glad to present our
```
 - it's space-separated. The first three columns are:
 - column1: the emission time of that line, in miliseconds. Only this number is used for the latency calculation
 - column2, column3: the beginning and end timestamp of the line in original audio. (Not relevant for this script, it can be anything.)
 - column4: This column starts either with a space, if the last line was to end with space, or with a character that has to be appended to the previous line. It's a character sequence emmitted by ASR together

3. **Usage:**

This is how to run the script with a tiny short demo input that is processed immediately.

- 1st commandline argument: the gold transcript filename
- standard input: the candidate 

```
$ python3 latency.py demo-data/tiny-gold.2022.acl-long.110.wav.ts.txt < demo-data/tiny-candidate.2022.acl-long.110.txt
Average Latency: 1.966727272727273 seconds
1.966727272727273
```

4. **Debug info:**

- optional 2nd commandline argument: "--debug", or "-d" or "debug" or anything to print the debug info

```
$ python3 latency.py demo-data/tiny-gold.2022.acl-long.110.wav.ts.txt --debug < demo-data/tiny-candidate.2022.acl-long.110.txt 
# char alignments:
# edit-operations \t gold \t asr \t latency diff or -1 if not available
# (note: sometimes there is a space that is is printed but not visible)
COPY	H	H	1.49
COPY	e	e	1.49
COPY	l	l	1.49
COPY	l	l	1.49
COPY	o	o	1.49
COPY	,	,	1.49
COPY	 	 	1.49
COPY	t	t	1.16
COPY	h	h	1.16
COPY	i	i	1.16
COPY	s	s	1.16
COPY	 	 	1.16
COPY	i	i	1.01
COPY	s	s	1.01
COPY	 	 	1.01
COPY	J	J	2.61
COPY	i	i	2.61
COPY	a	a	2.61
COPY	w	w	2.61
COPY	e	e	2.61
COPY	i	i	2.61
COPY	 	 	2.61
COPY	Z	Z	2.25
COPY	h	h	2.25
COPY	o	o	2.25
COPY	u	u	2.25
COPY	 	 	2.25
COPY	f	f	2.00
COPY	r	r	2.00
COPY	o	o	2.00
COPY	m	m	2.00
COPY	 	 	2.00
COPY	H	H	1.70
COPY	a	a	1.70
COPY	r	r	1.70
COPY	v	v	1.70
COPY	a	a	1.70
COPY	r	r	1.70
COPY	d	d	1.70
COPY	 	 	1.70
COPY	U	U	2.86
COPY	n	n	2.86
COPY	i	i	2.86
COPY	v	v	2.86
COPY	e	e	2.86
COPY	r	r	2.86
COPY	s	s	2.86
COPY	i	i	2.86
COPY	t	t	2.86
COPY	y	y	2.86
COPY	.	.	2.86
COPY	 	 	2.86
COPY	I	I	2.29
COPY	 	 	2.29
COPY	a	a	2.15
COPY	m	m	2.15
INS		 	-1
INS		v	-1
INS		e	-1
INS		r	-1
INS		y	-1
INS		 	-1
INS		g	-1
INS		l	-1
INS		a	-1
INS		d	-1
INS		 	-1
INS		t	-1
INS		o	-1
INS		 	-1
INS		p	-1
INS		r	-1
INS		e	-1
INS		s	-1
INS		e	-1
INS		n	-1
INS		t	-1
INS		 	-1
INS		o	-1
INS		u	-1
INS		r	-1
COPY	 	 	2.15
# char alignments ended.
# word alignments
# edit-operations, gold, asr, latency-diff or -1 if not available
COPY	Hello, 	Hello, 	1.49
COPY	this 	this 	1.16
COPY	is 	is 	1.01
COPY	Jiawei 	Jiawei 	2.61
COPY	Zhou 	Zhou 	2.25
COPY	from 	from 	2.00
COPY	Harvard 	Harvard 	1.70
COPY	University. 	University. 	2.86
COPY	I 	I 	2.29
SUB	am	am 	2.15
INS		very 	-1
INS		glad 	-1
INS		to 	-1
INS		present 	-1
SUB	 	our 	2.15
# word alignments ended.
# length of word_alignment: 15
# length of gold words: 10
# length of asr words: 15
Average Latency: 1.966727272727273 seconds
1.966727272727273
```

- standard output: the avarege word latency in seconds

```
1.966727272727273
```

## Demo with longer inputs

This demonstrates that a transcript of 11 minutes and 40 seconds is processed in 1 minute:

```
$ time python3 latency.py demo-data/gold.2022.acl-long.110.wav.ts.txt  < demo-data/candidate.2022.acl-long.110.txt 
Average Latency: 1.813456177192983 seconds
1.813456177192983

real	1m9,726s
user	1m8,197s
sys	0m1,468s
```

## Citation

Please cite the CUNI submission to the IWSLT 2025 Simultaneous Shared Task:

```
@inproceedings{machacek-and-polak-2025-simulaneous,
  title = {Simultaneous Translation with Offline Speech and LLM Models in CUNI
Submission to IWSLT 2025},
  author = {Macháček, Dominik and 
            Polák, Peter},
  editor = {Salesky, Elizabeth and Federico, Marcello and Anastasopoulos, Antonios},
  booktitle = {Proceedings of the 22nd International Conference on Spoken Language Translation (IWSLT 2025)},
  month = July,
  year = {2025},
  address = {Vienna, Austia (in-person and online)},
  publisher = {Association for Computational Linguistics},
  note = {To appear}
}
```

## Contact

Author: [Dominik Macháček](https://ufal.mff.cuni.cz/dominik-machacek), machacek@ufal.mff.cuni.cz

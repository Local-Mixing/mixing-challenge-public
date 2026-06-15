---
layout: post
title:  "2: Obfuscated Point Function"
date:   2026-05-24
---

{% include math.html %}

In this challenge, we have obfuscated a _point function_ -- that is, a function $$f$$ such that:

$$
f_{a,b}(x) = \begin{cases}
a &\text{if}\ x = b\\
b &\text{if}\ x = a\\
x & \text{otherwise}
\end{cases}
$$

When viewed as a permutation, $$f(\cdot)$$ comprises the single transposition $$(a\ b)$$.

Since reversible circuits can only compute _even_ permutations, it's not possible, strictly speaking, to encode a point function. Instead, we need an _ancilla_ wire, an extra wire which is effectively used as a temporary register. Inputs on ancilla wires do not effect the computation, and the outputs on those wires (generally) remain unchanged. Then, for some secret key $$k$$, we can build a circuit computing the permutation:

$$
(00\|k\ 01\|k)(10\|k\ 11\|k)
$$

where $$\cdot\|\cdot$$ represents bitstring concatenation. In this circuit, the ancilla is the MSB, and the active wire the second bit. Note that for these circuits, _four_ inputs do not map to themselves.

## The Plaintext Construction

One straightforward way to construct a circuit for $$f_{a,b}(x)$$ is to set $$b=a+2^{\lambda+1}$$: this way, we flip a single high-order bit when the lower-order bits match a secret key. The high-level approach is to XOR the input with the key, check if the lower wires are all zero, and XOR in the key again. Let $$z(x) := (x \overset ? = 0)$$ be the zero-check function. We compute

$$
f_{k,k+2^{\lambda+1}}(x)=z(x \oplus k) \| x
$$

Schematically:

![Point function construction](/assets/img/pf-constr.png){: width="50%"}

**TODO:** update this circuit to match the notation described above.

We implement $$z(x)$$ with a multi-control Toffoli gate. Such a gate flips its active pin only when all of its control pins are 1. We use the multi-control Toffoli construction of Barenco et al.[^1] to decompose this gate into two-input gates. Specifically, Section 7.1 gives a construction for a $$\lambda$$-control Toffoli in $$8(\lambda - 3)$$ two-control Toffoli gates. Next, we decompose each Toffoli into four copies of gate 57. Each of the $$\boxed{\oplus\neg k}$$ blocks can be represented with at most $$6\lambda$$ gates, so the final construction has $$44\lambda-96$$ gates.

[^1]: Barenco, Adriano, et al. "Elementary gates for quantum computation." _Physical review A_ 52.5 (1995): 3457. [arXiv](https://arxiv.org/abs/quant-ph/9503016v1)

As a concrete example, if we take $$k=100\in\mathbb F_2^8$$, our point function circuit would compute
$$
(100\ 356)(612\ 868)
$$. Here is such a ten-wire circuit:

```
215;840;516;215;517;651;216;617;516;651;157;671;326;157;814;912;419;923;926;419;
840;052;814;419;926;923;419;345;326;912;167;943;350;061;067;052;760;352;176;760;
347;054;178;064;164;061;350;345;354;056;943;327;054;350;945;947;347;165;164;061;
176;876;178;061;876;350;165;056;401;804;416;801;627;623;461;806;327;613;804;461;
631;632;406;641;416;641;406;143;056;263;941;150;062;063;056;870;270;278;870;054;
064;264;263;062;150;943;156;154;143;951;056;914;054;150;062;276;876;278;715;062;
275;715;264;915;150;215;427;876;056;647;427;275;627;571;537;647;731;517;573;731;
```

The following visualization plots gates left-to-right, with active pins in red and control pins notated by gray and black bars (we're not showing control polarity here). Wires travel horizontally but are left implicit for clarity.

![Small point function](/assets/img/small-pf.png)

## Test Circuits

Here is an _unobfuscated_ point function circuit on 32 wires that computes the cycle $$({*}0\|k\ \ {*}1\|k)$$, for $$k=123456789$$. The circuit has 1224 gates; inputs are over the lower 30 bits, and the ancilla and active wires are the first two.

```
05n;2hf;5nh;05h;0n5;5nh;0h5;aq5;hf3;2fh;2h3;hf3;23h;fl3;34l;jk7;7ek;ke7;j7k;ke7;7ek;aki;kin;aik;akn;kin;ank;l43;f3l;8sf;l43;34l;4o0;53q;o0t;40o;4ot;o0t;4to;pm1;1tm;mt1;p1m;mt1;1tm;mt6;6ot;q35;a5q;q35;53q;sfe;8fs;8se;sfe;8es;eq2;q2o;e2q;eqo;q2o;eoq;to6;m6t;gim;im8;gi8;gmi;im8;g8i;to6;6ot;p69;956;659;p96;659;956;5sc;b63;63o;b36;b6o;63o;bo6;cks;fb2;b2j;f2b;fbj;b2j;27l;fjb;hb6;b6i;h6b;hbi;b6i;hib;lp7;skc;5cs;skc;asi;cks;gcd;d9c;c9d;gdc;c9d;d9c;i4s;ko9;md1;d1f;m1d;mdf;d1f;mfd;s4i;ais;jao;aog;jag;joa;aog;jga;o97;k9o;ko7;o97;9mn;k7o;7pl;2l7;7pl;lp7;nfm;mfn;9nm;mfn;nfm;fa7;o56;p9c;9cd;p9d;pc9;9cd;pd9;9kt;q2j;qgl;gl1;qg1;qlg;gl1;q1g;1gr;s4i;56s;i4s;l8i;o5s;o65;56s;os5;rig;gir;1rg;gir;r3a;rcs;rig;g75;s3c;c3s;rsc;c3s;edc;obc;pc2;s1h;s3c;tak;kat;9tk;kat;m9a;t5i;tak;k68;u04;u08;v08;u0v;v04;v08;v12;v1h;v1s;s1h;h41;s2j;s2q;q2j;q3a;q3a;q3r;r3a;r3a;r41;r4h;h41;h5i;h5t;r4s;t58;t5i;t68;t6i;t6k;k68;k6i;k74;k75;k7g;g74;g75;g8i;g8l;g8s;l8i;l8s;l9a;l9h;l9m;m9a;m9h;ma7;maf;fa7;faj;fb7;fbc;fbo;maj;ob7;obc;oc2;ocd;ocp;pc2;pcd;pdc;pde;edc;ocp;fbo;maf;faj;l9m;g8l;k7g;g74;l8s;m98;m9h;h4s;h58;ob4;ob7;pca;pcd;q2f;t6k;h5t;k63;k6i;r4h;h4l;h4s;q3r;r36;r3a;s12;s2f;s2q;q2f;t58;q2t;v1s;s12;s1g;u0v;v04;v1d;v1g;v1s;s1g;g7e;s2q;q28;q2t;q36;q3a;q3r;r36;r3a;r4h;h4l;h5j;r4l;l8j;s2t;fas;t5j;h5t;t5j;t63;t6k;k63;k7e;k7g;g7e;ed5;g8j;g8l;k7t;l86;l8j;l98;l9g;l9m;m98;m9g;maf;fas;fb4;fb7;fbo;mas;ob4;ob7;oca;ocp;oct;pca;pct;pd5;pde;ed5;h5e;ocp;fbo;ob7;fao;maf;fao;mao;l9m;m9g;g7t;g86;g8l;k7g;g7t;l86;pct;k6p;s1d;s28;t5e;t6k;k6p;h4k;r4k;t6p;6fm;h5t;r4h;h4k;4gm;7hf;q3r;r3a;3ig;s2q;q28;8lb;t5e;5ka;em2;v1s;s1d;dn9;vf6;6fm;6g4;4gm;4h7;4hf;0s4;6gm;1j6;7hf;7i3;3ig;3j1;1j6;1k5;1ka;3j6;5ka;5kh;5l8;5lb;5li;6ft;6gl;7ig;8lb;8li;8m2;8me;aqo;em2;2pc;en9;end;dn9;9ol;do7;do9;9ol;9p2;2pc;2pf;2qa;2qi;2qo;9pc;aqi;aqo;arh;arq;crq;arc;crh;crq;cs0;0s1;0s4;0t9;cs1;cs4;4gl;4hq;dol;ut9;0tu;cs0;0s1;1kh;arc;2qa;aqi;aqp;crh;dn0;en0;ut9;9o7;9p2;2pf;8m2;9pf;do9;9o7;1j9;3j9;7hq;em2;end;8me;5l8;1k5;3j1;1j9;3i1;5kh;5ki;7i1;7i3;3i1;1jo;3iq;4h7;6g4;4gl;7hq;4g7;8li;9oc;dn0;0sg;dni;em2;2pq;emq;utp;vf6;6ft;6g4;4g7;4ht;6g7;7ht;4h7;7ht;7i3;3iq;3j1;1jo;1k5;1ki;1kl;3jo;5ki;5kl;5l3;5lm;7iq;1j7;8l3;5l8;8l3;8lm;8me;8mj;8mq;emj;emq;end;dni;dnl;do9;9oc;9p2;2pq;2qa;2qp;9pq;aqk;aqp;ar2;ar3;doc;cr3;arc;cr2;cr3;3j7;cs0;0s9;0sg;0tp;0tu;cs0;0s9;cs9;9os;9pn;csg;arc;cr2;2pn;2qa;2qk;9p2;2pn;4h2;aqk;4ga;6ga;do9;9os;dos;end;dnl;eni;enl;8me;5l8;1k5;3j1;1j7;5kl;7h2;8lm;emj;3ie;7i3;3ie;7ie;4h7;6g4;4ga;7h2;g6c;h15;i51;j3b;k4p;n0r;s7t;u0n;n0r;n15;n1h;h15;h2a;u0o;u0r;utp;v2a;vfm;vft;fao;h2v;rbh;v2a;v2n;v3b;v3c;v3j;j3b;j4k;j4p;k4h;k4p;k51;k5i;i51;i6c;i6g;g6c;g7s;g7t;edg;i6h;mcj;p9h;s72;s7t;s80;s8c;t80;s8t;t80;t8c;t97;t9h;t9p;p97;p9h;pa8;paf;fa8;fao;fbg;fbh;fbr;pao;rbg;rbh;rcj;rcj;rcm;mcj;mcj;mde;edg;mdg;rcm;fbr;mcb;mcj;j3c;j4h;paf;fa8;rbg;g6h;g72;t9p;p97;i5p;k5p;s8t;g7s;i6g;g67;g6h;k5i;i5e;i5p;j4k;k4h;h19;h2n;k4p;s72;s7p;t8c;t8f;v3j;h2v;j3c;j3r;v23;v2n;n0o;n19;n1h;h17;h19;u0n;n0o;n17;n19;n1h;h17;h19;h23;h2b;h2v;v23;v2b;v31;v3j;j3r;j4k;j4p;k42;k4p;k5e;k5i;i5e;i5j;i64;i67;i6g;g67;g7p;g7s;k5j;s7p;p9c;s8f;s8t;t8f;fag;t92;t9c;t9p;p9c;paf;fag;fag;fbo;fbq;pag;edp;v3r;rbo;fbr;rbo;rbq;rcb;rcg;rcm;mcb;mcg;mde;edp;mdp;p92;pag;rcm;fbr;mcg;dm8;paf;fag;g64;g7m;rbq;s7m;s8f;t8f;t9p;p92;s8t;g7s;i6g;g64;k5i;i5j;cia;j31;j42;j4k;k42;0kf;s7m;7pe;t8f;5rt;utm;v3j;h2v;j31;n1h;h19;1hq;9go;v2b;2fa;bj3;vf2;2fa;2g9;2go;2gs;9go;9gs;9h1;1hq;1ia;1ic;9hq;cia;cj3;cjb;bj3;3l8;bk0;0kf;0l3;0l8;3l8;3m8;3md;4s0;bkf;dm8;8nb;dn8;8nb;8nj;8o3;dnb;8od;bkc;vfa;aod;8oa;aod;ap7;7pe;7pg;7qt;ape;7qa;dm6;dnj;eqa;7qe;eqa;eqt;er5;5rt;5rt;5s0;5s4;4s0;0kc;0li;4sa;4tm;4tu;5s4;4sa;4sr;5sa;ao3;apg;er5;5rt;1i5;bj5;ert;ert;7qe;ap7;7pg;8oa;ao3;2fa;3li;3m6;dn8;3md;0l3;3li;8nj;bk0;0kc;0ks;ci5;cj5;cjb;1ic;bj5;bj5;ci5;1hc;9h1;1hc;1ho;9hc;2g9;9g3;9gs;cil;dm6;dmi;eqm;eqt;utf;utm;vf2;2fa;2g3;2g9;9g3;3lg;9h1;1ho;1ic;1if;1il;9ho;cil;cj5;cjb;bj5;5rt;bjf;bk0;0ks;0l3;0lg;3lg;3md;3mi;3mt;8n0;bks;dmi;dmt;dn0;dn8;8n0;8os;vfa;aos;8oa;7p8;aoc;aos;ap7;7p8;7pf;7qe;7ql;7qm;ap8;8oc;apf;eql;eqm;er5;5rt;5s4;4sg;4sr;4tf;4tu;5s4;4sg;5r4;5sg;5sr;er4;er5;5r4;ert;7qe;ap7;7pf;8n7;8oa;aoc;cif;cjf;dn7;dn8;3md;8n7;1h8;9h8;dmt;eql;0le;3le;0l3;3le;0k3;2g3;9g3;bk0;04d;0k3;4dl;04l;0d4;4dl;0l4;bk3;cjb;1ic;9h1;1h8;2g9;9g3;bfr;bjf;cif;dn6;6cn;dq1;12q;g07;nc6;d6n;nc6;6cn;q21;d1q;q21;12q;2ml;mli;2lm;2mi;mli;2im;427;27j;42j;472;27j;4j2;2p5;51p;il3;37l;l73;i3l;l73;37l;7i0;0i7;g39;9q3;3q9;g70;0i7;7i0;b0c;cj0;0jc;bc0;0jc;cj0;g93;3q9;9q3;gcd;drc;crd;gdc;crd;drc;p15;25p;p15;51p;8pf;al5;l5f;a5l;alf;l5f;afl;pfq;8fp;8pq;pfq;8qp;utf;frh;bfh;brf;frh;bhf;ebf;bf5;eb5;efb;bf5;e5b;fo8;hkt;kt7;hk7;htk;kt7;h7k;mh9;o81;f8o;fo1;o81;f1o;gcf;cfq;gcq;gfc;cfq;5cl;gqc;l6c;c6l;5lc;c6l;l6c;poi;ijo;oji;pio;kfp;fpg;kfg;kpf;fpg;h9f;kgf;m9h;mhf;h9f;mfh;oji;ijo;jb0;b0g;j0b;jbg;b0g;02n;jgb;ni2;2in;0n2;2in;net;ni2;o6t;6t3;o63;ot6;6t3;o36;pgo;goh;pgh;pog;goh;phg;qko;kot;qkt;qok;kot;qtk;sqr;r2q;q2r;srq;bjs;q2r;r2q;sij;jis;bsj;jis;sij;t3e;e3t;nte;e3t;t3e;
```

Running the above circuit through our randomized compressor, we get a slightly shorter, but functionality equivalent, 762-gate circuit:

```
qpc;p9c;a20;02f;a20;2hf;hf3;02f;efs;0hf;4o0;o0t;8sf;es8;8se;40o;4ot;64t;af2;2h3;2fh;esf;efs;8se;es8;gim;v08;im8;edc;u0v;o0t;jao;kjo;b6t;gi8;gmi;hf3;f8g;f8i;64t;im8;l8i;23h;hdm;g8i;aog;joa;kjo;jag;q9c;9cd;k6i;ko6;4to;v04;v08;aog;o56;56s;k6o;o65;o5s;56s;s12;pc2;qpc;p9d;pc9;mdh;jga;q2j;v1s;s12;os5;r3a;s2q;s2j;q3r;f8g;mfh;hdf;mhd;mhf;g75;hdm;mdf;fa7;9cd;m9a;pd9;b6t;b4t;obc;t58;h4s;r4h;h5t;t6k;k7g;g8l;l9m;maf;l8s;fbo;ocp;g74;ob7;maj;faj;l9a;fb7;fbc;fa7;m98;m9a;ma7;k75;k74;l98;g75;pdc;obc;pde;ocp;u04;q2j;fbo;maf;ob7;l8s;l9m;g8l;k7g;l8i;t6k;g74;h5t;r4h;q3r;h4l;h4s;t58;k6i;k63;ob4;s12;q2t;s2t;s2q;v1s;u0v;v1s;s2t;s2q;q28;q3r;r4h;h5j;q2t;t5j;h5t;t6k;t63;v04;s12;r4l;fas;h4l;l8j;t5j;g7t;k7g;pc2;pca;faj;g8l;l9m;maf;fbo;ocp;pde;edc;pdc;h5e;ocp;fbo;ob4;pca;maf;fas;l9m;g8l;l8j;k7g;m98;g7t;t5e;t6k;t63;h5t;r4h;7hf;k63;6fm;rg4;4gm;t5e;em2;vft;s1d;u08;v1d;r4g;q3r;r3a;3ig;vf6;6gm;6gl;6g4;4hf;4gm;6fm;s28;s2q;v1s;s1d;4h7;q28;7i3;3k1;7hq;3iq;7iq;8li;7hf;7ig;6ft;dn9;5kh;3ig;31j;v12;1k5;31k;5l8;8me;8m2;em2;end;2pf;aqo;crq;0s4;dn9;en9;9ol;do9;9p2;2qa;arc;cs0;utp;0tu;0tp;cs0;0s4;dn0;4hq;4gl;arc;2qa;aqp;aqo;9p2;do9;2pf;em2;9ol;end;en0;1j9;8m2;8me;5l8;1k5;dnl;dn0;3j1;3j9;7i3;4h7;6g4;vf6;6ft;6g4;crq;8lm;8li;emj;1jo;5kl;5kh;7ht;1j9;9oc;7hq;4h7;7i3;3j1;7iq;1k5;em2;2pq;5l8;8me;3jo;3iq;end;do9;9os;9oc;9p2;2pn;doc;2qa;cr3;arc;cs0;0tp;0tu;u0o;utp;u0r;cs0;arc;cr3;3jo;dos;2pn;2qa;9p2;2pq;aqp;v2a;do9;end;dnl;8me;9os;5l8;8lm;emj;1k5;3j1;3ie;7i3;1jo;5kl;7ie;4h7;3ie;j3b;7ht;h15;s7t;vfm;n0r;6g4;i51;u0n;n15;n1h;h2v;h15;v3c;v3j;v2a;v3b;j4g;4gl;h2a;j4k;k5i;i6c;k4h;n0r;v2n;p9h;jg4;g6c;i6g;g7s;g7t;st8;s72;g6c;edg;s8c;j3b;mcj;rbh;fao;s9t;s7t;t9p;paf;fbr;rcm;p97;mde;edg;st9;p9h;t9h;t97;t8c;mdg;g6h;g72;rcm;mcb;mcj;j3c;fbr;paf;fao;t9p;s8t;t8f;g7s;j4h;rbh;p97;i6g;g6h;i6h;g67;k5i;j4k;k4p;k4h;h2n;h19;v3j;j3r;h2v;v2n;n1h;n19;n0o;u0n;n1h;v2b;j3c;h2v;v3r;s7p;v3j;j3r;i5e;i51;j4k;k5i;i67;k42;i64;k4p;s72;i6g;g7s;g7p;j4p;i5j;s7p;p9c;rbo;t8c;g67;s8f;s8t;t8f;fag;t9p;paf;fbr;t92;t9c;k5e;pag;i5e;rcm;p9c;edp;mdp;mde;rcm;mcb;s7m;fbr;edp;p92;pag;dm8;paf;t9p;sut;fag;eqt;g7m;st8;utm;sut;g7s;g64;i6g;g64;s7m;n0o;k5j;k5i;i5j;j4k;vj4;cia;k42;0kf;j42;vj3;rbo;v4j;h2v;v2b;bj3;p92;n1h;h19;2fa;1ho;vf2;5rt;9gs;2g9;9h1;1ic;cjb;bj3;cia;cj3;3l8;1ia;vfa;7pf;2fa;bk0;0l8;0l3;3m8;3lg;3m6;3l8;0kf;3md;bkf;dnb;4s0;dm8;8nb;dn8;aod;8oa;ap7;7qe;er5;5s4;4tm;4tu;utf;utm;5s4;4sg;4s0;0kc;0lg;er5;7qe;ap7;5rt;8oa;eql;aod;2fa;dm6;dnb;dn8;3md;0l3;8nb;bk0;bkc;0ks;0kc;ci5;1i5;bj5;cj5;cjb;1ic;9h1;cil;2g9;eqt;vf2;2ml;2g9;9h1;1il;9ho;9gs;1ho;vfa;2fa;ci5;1ic;dmt;dm6;cjb;bk0;0l3;cil;bjf;bks;3lg;0ks;0lg;3md;cj5;bj5;aoc;5rt;do8;d8n;8oa;d8o;ap7;7qe;er5;5s4;4tf;4tu;5s4;er5;7qe;ap7;8oa;eql;d8n;da8;a8f;3le;da8;3md;0l3;3le;0k3;9g3;2g3;aoc;4sg;0le;bk3;dmt;bk0;04d;cjb;4dl;o9i;mli;0d4;dml;2dl;bfu;jft;cif;0k3;8pf;a8f;8af;bjf;utf;o1i;1if;91i;7pf;pfq;apf;8af;jft;cjf;1ic;o9i;91h;hkt;kt7;cif;5rt;bfu;ebf;bf5;eb5;efb;9i1;2g9;hk7;htk;pfq;f5e;f5b;bf5;e5b;2gs;9g3;kt7;gkp;f5e;h7k;kfp;fpg;04l;4dl;dml;2dl;0l4;mli;4ml;mh9;jb0;gfp;gkp;b0g;kfg;kpf;fpg;h9f;mhf;m9h;qfm;j0b;jbg;kgf;pgo;qfh;b0g;h9f;mfh;jgb;goh;pog;pgh;qfm;goh;phg;
```

And an obfuscated version of the same:

```
# TODO
```

All three of these circuits compute the permutation:

$$
\begin{gather}
(k=123456789\quad k+2^{31})\\
(k+2^{32}\quad k+2^{31}+2^{32})
\end{gather}
$$

That is, bit 31 flips on the correct key, and remains unchanged otherwise.

To generate your own point functions, see the file `point_function.rs`.

## The challenges

The obfuscated point function below computes the cycle $$({*}0\|k\ \ {*}1\|k)$$, for $$k\in\mathbb F_2^{128}$$. Find $$k$$. (The circuit has 132 wires, so you probably won't have enough time to brute-force all possible inputs.)

```
# TODO
```

### A Distinguishing Challenge

If finding the point seems intractable, this challenge may be easier. Let $$f_k(x)$$ be a point function of key $$k$$, as defined above. Then, $$f_k(f_k(x))=x$$ (that is, the concatenation of the two circuits) equals the identity, while $$f_{k'}(f_k(x))$$ (the concatentation of two _different_ point functions) does not.

[This zip archive](#) contains 256 obfuscated circuits which are each the concatenation of two point functions. With probabily $$1/2$$, we either set $$k'=k$$ or $$k'\neq k$$ (so about half of them are the identity circuit, and the rest compute another point function). For each circuit $$C_i$$, let $$b_i = \mathbb{1}_{[k'=k]}$$ (one if the keys are the same and zero if they are different). The solution to this challenge is the bitstring

$$
b_{255}b_{254}\dots b_1b_0
$$
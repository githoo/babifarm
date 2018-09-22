+++
categories = ["技术"]
date = "2017-09-22T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "core dump"

+++


## core dump 定位分析

core dump 日志分析  core.212160


###1、定位core发生的位置： 
```
gdb /export/servers/model/crf/so_lib/libCRFPP.so core.212160
(gdb) bt
#6  <signal handler called>
#7  0x00007f269c2a34be in CRFPP::TaggerImpl::forwardbackward() () from /tmp/crfpp-java-0.57-fcac54dc-13d1-48a4-b76d-b2941f15da0d-libCRFPP.so
#8  0x00007f269c2aad92 in CRFPP::TaggerImpl::parse() () from /tmp/crfpp-java-0.57-fcac54dc-13d1-48a4-b76d-b2941f15da0d-libCRFPP.so
#9  0x00007f269c2a18f0 in Java_org_chasen_crfpp_CRFPPJNI_Tagger_1parse_1_1SWIG_10 ()
   from /tmp/crfpp-java-0.57-fcac54dc-13d1-48a4-b76d-b2941f15da0d-libCRFPP.so
```

###2、定位dump的细节
```
objdump  -Cd /export/servers/model/crf/so_lib/libCRFPP.so > tt.txt 

cat tt.txt|grep "forwardbackward"
[root@ckespam-e31e7701-2927318961-21969 App]# cat tt.txt|grep "forwardbackward"                                   
0000000000010848 <CRFPP::TaggerImpl::forwardbackward()@plt>:
0000000000027f40 <CRFPP::TaggerImpl::forwardbackward()>:
   27f5a:       0f 84 98 01 00 00       je     280f8 <CRFPP::TaggerImpl::forwardbackward()+0x1b8>
   27f7a:       7e 65                   jle    27fe1 <CRFPP::TaggerImpl::forwardbackward()+0xa1>
   27f95:       74 2f                   je     27fc6 <CRFPP::TaggerImpl::forwardbackward()+0x86>
   27fbc:       77 e2                   ja     27fa0 <CRFPP::TaggerImpl::forwardbackward()+0x60>
   27fdf:       7c af                   jl     27f90 <CRFPP::TaggerImpl::forwardbackward()+0x50>
   27fe6:       78 56                   js     2803e <CRFPP::TaggerImpl::forwardbackward()+0xfe>
   28005:       74 27                   je     2802e <CRFPP::TaggerImpl::forwardbackward()+0xee>
   2802c:       77 e2                   ja     28010 <CRFPP::TaggerImpl::forwardbackward()+0xd0>
   2803c:       79 c2                   jns    28000 <CRFPP::TaggerImpl::forwardbackward()+0xc0>
   2804b:       0f 84 a7 00 00 00       je     280f8 <CRFPP::TaggerImpl::forwardbackward()+0x1b8>
   2807d:       76 79                   jbe    280f8 <CRFPP::TaggerImpl::forwardbackward()+0x1b8>
   28092:       74 74                   je     28108 <CRFPP::TaggerImpl::forwardbackward()+0x1c8>
   280b0:       77 be                   ja     28070 <CRFPP::TaggerImpl::forwardbackward()+0x130>
   280ef:       77 8e                   ja     2807f <CRFPP::TaggerImpl::forwardbackward()+0x13f>
   2810c:       e9 5f ff ff ff          jmpq   28070 <CRFPP::TaggerImpl::forwardbackward()+0x130>
   28628:       e8 1b 82 fe ff          callq  10848 <CRFPP::TaggerImpl::forwardbackward()@plt>
   2b04c:       e8 f7 57 fe ff          callq  10848 <CRFPP::TaggerImpl::forwardbackward()@plt>
```

###3、定位行号  
```
addr2line  -i -e /export/servers/model/crf/so_lib/libCRFPP.so   0x地址

[root@ckespam-e31e7701-2927318961-21969 App]# addr2line  -i -e /export/servers/model/crf/so_lib/libCRFPP.so  0x28628
/export/fisher/crfpp-master/tagger.cpp:571
[root@ckespam-e31e7701-2927318961-21969 App]# addr2line  -i -e /export/servers/model/crf/so_lib/libCRFPP.so  0x2b04c
/export/fisher/crfpp-master/tagger.cpp:675
```



###4、源码定位

```
bool TaggerImpl::parse() {
  CHECK_FALSE(feature_index_->buildFeatures(this))
      << feature_index_->what();

  if (x_.empty()) {
    return true;
  }
  buildLattice();
  if (nbest_ || vlevel_ >= 1) {
    forwardbackward();
  }
  viterbi();
  if (nbest_) {
    initNbest();
  }

  return true;
}

 
double TaggerImpl::gradient(double *expected) {
  if (x_.empty()) return 0.0;

  buildLattice();
  forwardbackward();
  double s = 0.0;

  for (size_t i = 0;   i < x_.size(); ++i) {
    for (size_t j = 0; j < ysize_; ++j) {
      node_[i][j]->calcExpectation(expected, Z_, ysize_);
    }
  }

  for (size_t i = 0;   i < x_.size(); ++i) {
    for (const int *f = node_[i][answer_[i]]->fvector; *f != -1; ++f) {
      --expected[*f + answer_[i]];
    }
    s += node_[i][answer_[i]]->cost;  // UNIGRAM cost
    const std::vector<Path *> &lpath = node_[i][answer_[i]]->lpath;
    for (const_Path_iterator it = lpath.begin(); it != lpath.end(); ++it) {
      if ((*it)->lnode->y == answer_[(*it)->lnode->x]) {
        for (const int *f = (*it)->fvector; *f != -1; ++f) {
          --expected[*f +(*it)->lnode->y * ysize_ +(*it)->rnode->y];
        }
        s += (*it)->cost;  // BIGRAM COST
        break;
      }
    }
  }

  viterbi();  // call for eval()

  return Z_ - s ;
}

void TaggerImpl::forwardbackward() {
  if (x_.empty()) {
    return;
  }

  for (int i = 0; i < static_cast<int>(x_.size()); ++i) {
    for (size_t j = 0; j < ysize_; ++j) {
      node_[i][j]->calcAlpha();
    }
  }

  for (int i = static_cast<int>(x_.size() - 1); i >= 0;  --i) {
    for (size_t j = 0; j < ysize_; ++j) {
      node_[i][j]->calcBeta();
    }
  }

  Z_ = 0.0;
  for (size_t j = 0; j < ysize_; ++j) {
    Z_ = logsumexp(Z_, node_[0][j]->beta, j == 0);
  }

  return;
}
```

定位到forwardbackward();函数产生core地方，node_[i][j] 可能为空导致core dump。



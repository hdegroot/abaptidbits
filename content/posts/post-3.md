---
title: "Post 3"
date: 2022-01-31T21:31:23-08:00
draft: false
---

This is a test.



```abap

REPORT  ZFI_FIX_1KEK                            .


  TABLES:
    bkpf.


  DATA: it_bkpf TYPE STANDARD TABLE OF bkpf
                WITH HEADER LINE.

  DATA: it_bseg TYPE STANDARD TABLE OF bseg
              WITH HEADER LINE.

  DATA:
    v_count1 TYPE i,   "Number of lines with a profit center
    v_count2 TYPE i.   "Number of vendor and/or customer lines

  SELECT-OPTIONS:
    sbukrs FOR bkpf-bukrs,
    sgjahr FOR bkpf-gjahr,
    sbelnr FOR bkpf-belnr.



  SELECT * FROM bkpf INTO CORRESPONDING FIELDS OF TABLE it_bkpf
           WHERE bukrs IN sbukrs
             AND gjahr IN sgjahr
             AND belnr IN sbelnr.


  LOOP AT it_bkpf.

    CLEAR:
      it_bseg,
      it_bseg[].

    SELECT * FROM bseg INTO CORRESPONDING FIELDS OF TABLE it_bseg
             WHERE bukrs = it_bkpf-bukrs
               AND belnr = it_bkpf-belnr
               AND gjahr = it_bkpf-gjahr.

*   Mark document as "special" for 1KEK if there are
*   no lines with a profit center in the document
    v_count1 = 0.
    LOOP AT it_bseg WHERE prctr <> SPACE.
      v_count1 = v_count1 + 1.
    ENDLOOP.

    IF v_count1 = 0.

        WRITE: / it_bkpf-bukrs,
                 it_bkpf-belnr,
                 it_bkpf-gjahr,
                 it_bkpf-xblnr,
                 it_bkpf-bktxt,
                 it_bkpf-bvorg,
                 'No profit center in document.'.

        UPDATE rf048 SET blgchr = '1'
                  WHERE bukrs = it_bkpf-bukrs AND
                        belnr = it_bkpf-belnr AND
                        gjahr = it_bkpf-gjahr.
    ENDIF.


*   Mark document as "special" for 1KEK if there are
*   multiple vendor and/or customer lines in the document
    v_count2 = 0.
    LOOP AT it_bseg WHERE koart = 'K' OR koart = 'D'.
      v_count2 = v_count2 + 1.
    ENDLOOP.

    IF v_count2 >= 2.
        WRITE: / it_bkpf-bukrs,
                 it_bkpf-belnr,
                 it_bkpf-gjahr,
                 it_bkpf-xblnr,
                 it_bkpf-bktxt,
                 it_bkpf-bvorg,
                 'Multiple vendor and/or customer lines in document.'.

        UPDATE rf048 SET blgchr = '1'
                  WHERE bukrs = it_bkpf-bukrs AND
                        belnr = it_bkpf-belnr AND
                        gjahr = it_bkpf-gjahr.
    ENDIF.


  ENDLOOP.

```

This is normal text.

And here is some more text.

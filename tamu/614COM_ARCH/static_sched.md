Static scheduling
---

Loop:   L.D     F0, 0(R1)
        L.D     F1, -8(R1)
        L.D     F2, -16(R1)
        L.D     F3, -24(R1)
        ADD.D   F4, F0, F2
        ADD.D   F5, F1, F2
        ADD.D   F6, F2, F2
        ADD.D   F7, F3, F2
        S.D     F4, 0(R1)
        S.D     F5, -8(R1)
        S.D     F6, -16(R1)
        S.D     F7, -24(R1)
        DADDIU  R1, R1, -32
        BNE     R1, R2, Loop
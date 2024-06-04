# Task-5
/*----------------------------------------------------------------------
  This function performs a 3 x 3 convolution on the active image. The 
  kernel array is passed externally. Edges are added (doubly weighted)
  for the computation. 
----------------------------------------------------------------------*/
void CImagrDoc::Convl(float k1, float k2, float k3,
                       float k4, float k5, float k6,
                       float k7, float k8, float k9)
{
    int *p;                        /* Image ptr */
    unsigned long i, j, nx, ny;
    int *m1, *m2, *m3;            // Pointers to buffers to free()
    int *old_r1, *r1, *r2, *r3; /* Cycling pointers to rows */
    float s, fsum;
    int t;
    byte r, g, b;

    nx = m_image.GetWidth();
    ny = m_image.GetHeight();
    p = (int *) m_image.GetBits();    // Ptr to bitmap

    /* Allocate row buffers */
    if (!(m1 = (int *) malloc((nx+2) * sizeof(*m1)))) {
        fMessageBox("Error - " __FUNCTION__, MB_ICONERROR, "malloc() error m1");
        return;
    }
    if (!(m2 = (int *) malloc((nx+2) * sizeof(*m2)))) {
        fMessageBox("Error - " __FUNCTION__, MB_ICONERROR, "malloc() error m2");
        free(m1);
        return;
    }
    if (!(m3 = (int *) malloc((nx+2) * sizeof(*m3)))) {
        fMessageBox("Error - " __FUNCTION__, MB_ICONERROR, "malloc() error m3");
        free(m1);
        free(m2);
        return;
    }
    r1 = m1;
    r2 = m2;
    r3 = m3;

    // Initialize rows
    memcpy_s(&r1[1], nx * sizeof(int), p, nx * sizeof(int));
    r1[0] = r1[1];                      /* Doubly weight edges */
    r1[nx+1] = r1[nx];

    /* Start r2 = r1 (doubly weight 1st row) */
    memcpy_s(r2, (nx+2) * sizeof(int), r1, (nx+2) * sizeof(int));

    // Calc. sum of kernel
    fsum = k1 + k2 + k3 + k4 + k5 + k6 + k7 + k8 + k9;
    if (fsum == 0) 
        fsum = 1;            // Avoid div. by 0
    else
        fsum = 1/fsum;        // Invert so can mult. 

    OnDo();        // Save image for Undo

    BeginWaitCursor(); 
    switch (m_image.ptype) {
        case GREY:
            for (j = 1; j <= ny; j++, p += nx) {
                if (j == ny) {                /* Last row */
                    r3 = r2;                /* Last row doubly weighted */
                }
                else {     /* Read next row (into the 3rd row) */
                    memcpy_s(&r3[1], nx * sizeof(int), p + nx, nx * sizeof(int));
                    r3[0] = r3[1];            /* Doubly weight edges */
                    r3[nx+1] = r3[nx];
                }

                for (i = 0; i < nx; i++) {
                    s = k1 * (float)RED(r1[i]) 
                      + k2 * (float)RED(r1[i+1])
                      + k3 * (float)RED(r1[i+2]) 
                      + k4 * (float)RED(r2[i])
                      + k5 * (float)RED(r2[i+1])
                      + k6 * (float)RED(r2[i+2])
                      + k7 * (float)RED(r3[i])
                      + k8 * (float)RED(r3[i+1])
                      + k9 * (float)RED(r3[i+2]);

                    t = NINT(s * fsum);
                    r = (byte)THRESH(t);

                    p[i] = RGB(r, r, r);      
                }

                /* Cycle row pointers */
                old_r1 = r1;    // To save addr. for r3
                r1 = r2;
                r2 = r3;
                r3 = old_r1;
            }
            break;
    }
    EndWaitCursor();

    free(m1);                   
    free(m2);
    free(m3);                

    ChkData();                // Re-check range
    SetModifiedFlag(true);    // Set flag
    UpdateAllViews(NULL);    // Still needed even though called by ChkData()
}

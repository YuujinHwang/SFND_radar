# Radar Target Generation and Detection

### FMCW Waveform Design

Using the given system requirements, design a FMCW waveform. Find its Bandwidth(B), chirp time(Tchirp) and slop of the chirp

With the speed of light 3e+9, range resolution 1, and given radar freq. 77GHz

- B : 1.5 e+9
- Tchrip : 7.333e-6
- Slope : 2.0455 e+13

### Simulation Loop
Simulate Target movement and calculate the beat or mixed signal for every timestamp.

    r_t(i) = r + v*t(i);
    
    tau(i) = 2*r_t(i)/c;
    
    Tx(i) = cos(2*pi*(fc*t(i) + slope/2*t(i)^2));
    Rx (i)  = cos(2*pi*(fc*(t(i)-tau(i)) + slope/2*(t(i)-tau(i))^2));
    
    Mix(i) = Tx(i)*Rx(i);

### Range FFT(1st FFT)

<img src = fig1.png>

### 2D CFAR
Implement the 2D CFAR process on the output of 2D FFT operation

With range and doppler axis, raw 2D FFT output is like below.
<img src = fig2.png>
After applying CFAR with parameters

    Tr = 15;
    Td = 5;
    
    Gr = 5;
    Gd = 1;

    offset = 15;
- Raw signal shows about 20dB gain than noise
- Higher offset can make peak much sharp, but that might cause detection failure
- Large gaurd cell window makes the detected peak smoother (multiple cells detected)

Threshold(noise_level + offset) is determined average of cells of interest. Cells with value under threshold are suppressed to 0.

    for i = 1+Tr+Gr:Nr/2-Tr-Gr
        for j  = 1+Td+Gd:Nd-Td-Gd
            noise_level = pow2db((sum(db2pow(RDM(i-Tr-Gr:i+Tr+Gr, j-Td-Gd:j+Td+Gd)),'all') - sum(db2pow(RDM(i-Gr:i+Gr, j-Gd:j+Gd)),'all'))/((2*Gr+2*Tr+1)*(2*Gd+2*Td+1)-(2*Gr+1)*(2*Gd+1)));
            if (RDM(i,j) < noise_level + offset)
                RDM(i,j) = 0;
            else
                RDM(i,j) = 1;
            end
        end
    end

And the cells we cannot perform CFAR, also set as 0.

    RDM(1:Tr+Gr,:) = 0;
    RDM(end-(Tr+Gr):end,:) = 0;
    RDM(:, 1:Td+Gd) = 0;
    RDM(:, end-(Td+Gd):end)=0;

Detection heat map becomes:
<img src = fig3.png>

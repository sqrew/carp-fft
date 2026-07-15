# Examples

Here are some common ways to use the modular `carp-fft` library.

## Setup

First, load the required modules in your Carp entry point:

```clojure
(load "path/to/carp-fft/complex.carp")
(load "path/to/carp-fft/fft.carp")
(load "path/to/carp-fft/window.carp")
(load "path/to/carp-fft/spectrum.carp")
(load "path/to/carp-fft/convolution.carp")

(use Complex)
(use FFT)
(use Window)
(use Spectrum)
(use Convolution)
```

## Basic Complex Arithmetic

Create and manipulate complex numbers using the `Complex` module.

```clojure
(defn main []
  (let [c1 (Complex.init 3.0 4.0)          ; 3 + 4i
        c2 (Complex.init 1.0 -2.0)         ; 1 - 2i
        c-sum (Complex.+ &c1 &c2)          ; 4 + 2i
        c-prod (Complex.* &c1 &c2)         ; 11 - 2i
        mag (Complex.abs &c1)]             ; 5.0
    (do
      (println* "Sum: " (str &c-sum))
      (println* "Product: " (str &c-prod))
      (println* "Magnitude: " mag))))
```

## 1D FFT and IFFT

Perform a Fast Fourier Transform and reconstruct the original signal with the inverse FFT.

```clojure
(defn main []
  ;; Create a simple signal (e.g., 8-point sine wave + DC component)
  (let [signal [(Complex.init 1.0 0.0) (Complex.init 1.707 0.0)
                (Complex.init 2.0 0.0) (Complex.init 1.707 0.0)
                (Complex.init 1.0 0.0) (Complex.init 0.293 0.0)
                (Complex.init 0.0 0.0) (Complex.init 0.293 0.0)]
        ;; The non-try API panics on size mismatches, making it cleaner to use
        spectrum (FFT.fft &signal)
        reconstructed (FFT.ifft &spectrum)]
    (do
      (println* "Spectrum:")
      (for [i 0 8]
        (println* "  Freq " i ": " (str (Array.unsafe-nth &spectrum i))))
      (println* "Reconstructed matches original: "
                (Complex.approx (Array.unsafe-nth &reconstructed 1)
                                (Array.unsafe-nth &signal 1)
                                0.0001)))))
```

## Real FFT with Windowing

Apply a window function (like Hann or Flat Top) to a real-valued signal, perform a real-to-complex transform, and reconstruct.

```clojure
(defn main []
  (let [size 1024
        ;; 1. Generate window coefficients
        window (Window.hann size)
        ;; 2. Create sample real-valued signal (e.g., 1024-point sine)
        signal (Array.replicate size &0.5)]
    (do
      ;; 3. Apply window in-place
      (Window.apply-real! &signal &window)
      ;; 4. Run real FFT to get one-sided spectrum
      (let [spectrum (FFT.rfft-one-sided &signal)]
        (do
          (println* "One-sided spectrum length: " (Array.length &spectrum)) ; 513
          ;; 5. Reconstruct back to time domain
          (let [reconstructed (FFT.irfft &spectrum)]
            (println* "Reconstructed real signal length: " (Array.length &reconstructed))))))))
```

## Power and dB Spectra Analysis

Perform spectral analysis using the `Spectrum` module to inspect magnitudes or power in decibels.

```clojure
(defn main []
  (let [signal [(Complex.init 3.0 4.0) (Complex.init 0.0 -5.0)]
        mags (Spectrum.magnitude &signal)    ; [5.0, 5.0]
        power (Spectrum.power &signal)       ; [25.0, 25.0]
        db-vals (Spectrum.db &signal)]        ; dB scaled
    (do
      (println* "Magnitudes: " @(Array.unsafe-nth &mags 0))
      (println* "Power: " @(Array.unsafe-nth &power 0))
      (println* "dB (0): " @(Array.unsafe-nth &db-vals 0)))))
```

## Linear Convolution and Autocorrelation

Filter signals in the time-domain using high-performance frequency-domain convolution.

```clojure
(defn main []
  (let [a [1.0 2.0 3.0]
        b [0.0 1.0 0.5]
        ;; Performs fast linear convolution using RFFT/IRFFT internally
        conv (Convolution.convolve &a &b)              ; [0.0, 1.0, 2.5, 4.0, 1.5]
        ;; Auto-correlation (cross-correlation with itself)
        acorr (Convolution.autocorrelate &a)]           ; [2.0, 5.0, 2.0]
    (do
      (println* "Convolution length: " (Array.length &conv))
      (println* "Autocorrelation at lag 0 (energy): " @(Array.unsafe-nth &acorr 1)))))
```

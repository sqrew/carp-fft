# carp-fft

A robust, double-precision Fast Fourier Transform (FFT) and Digital Signal Processing (DSP) library for the [Carp language](https://github.com/carp-lang/Carp).

## Architecture

The library is separated into focused modules for easier extension and maintainability:

1. **Complex Numbers (`complex.carp`):** Loaded via dependency [carp-complex-numbers](https://github.com/sqrew/carp-complex-numbers), providing double-precision complex arithmetic (`+`, `-`, `*`, `/`, `abs`/`norm`, conjugate, exponentiation, trigonometric functions, etc.).
2. **Main Transforms (`fft.carp`):**
   - 1D FFT & IFFT (`fft`, `ifft`, `try-fft`, `try-ifft`) using a Cooley-Tukey Radix-2 decimation-in-time algorithm.
   - Real FFT (`rfft`, `rfft-one-sided`) and Complex-to-Real IFFT (`irfft`) which takes a one-sided spectrum (size $N/2 + 1$) and automatically reconstructs Hermitian symmetry before transforming.
   - 2D FFT & IFFT (`fft2d`, `ifft2d`, `try-fft2d`, `try-ifft2d`) via row-column decomposition.
   - Signal utilities like zero-padding (`pad-zero`, `pad-zero-complex`) and zero-frequency shifting (`fftshift`, `ifftshift`).
3. **Signal Windowing (`window.carp`):** Common window generators (Hann, Hamming, Blackman, Rectangular, Bartlett/Triangular, Gaussian, and Flat Top) with in-place multipliers (`apply!`, `apply-real!`).
4. **Spectral Analysis (`spectrum.carp`):** Functions to calculate magnitude, power, natural log, and decibel (dB) log-magnitude spectra (`magnitude`, `power`, `log-spectrum`, `db`).
5. **Signal Convolution (`convolution.carp`):** High-performance spectral domain operations: circular convolution (`circular-convolve`), linear convolution (`convolve`), cross-correlation (`cross-correlate`), and autocorrelation (`autocorrelate`).

## Installation

Add the library files to your project:

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

## Running Tests

Run the comprehensive test suite (verifying DSP behaviors like Impulse, Alternating/Nyquist spikes, Parseval's theorem, convolutions, and roundtrips):

```bash
carp -x test/fft_test.carp
```

## Examples

See [examples.md](examples.md) for usage examples.

## License

MIT

# OFDM-based-communication
OFDM-based communication system based on conditions- AWGN, Rayleigh, and Rician fading with (BER)

clc; 
clear; 
close all;

% Basic OFDM Parameters
N = 64;               % Number of subcarriers
CP = N/4;            % Cyclic Prefix Length
M = 4;               % Modulation Order (QPSK)
numSymbols = 1000;   % Number of OFDM Symbols

% Generating random data
data = randi([0 M-1], N*numSymbols, 1);

% Modulation (QPSK)
modData = pskmod(data, M, pi/4);

% Reshape for OFDM transmission
txData = reshape(modData, N, numSymbols);

% Applying IFFT
txIFFT = ifft(txData, N);

% Adding Cyclic Prefix
txSignal = [txIFFT(end-CP+1:end, :); txIFFT];

% Convert to serial for transmission
txSerial = txSignal(:);

SNR_dB = 10;  % Signal-to-Noise Ratio in dB
rxSerial_AWGN = awgn(txSerial, SNR_dB, 'measured');

rayleighChan = comm.RayleighChannel('SampleRate', 1e6, 'PathDelays', [0 2e-6], 'AveragePathGains', [0 -3]);
rxSerial_Rayleigh = rayleighChan(txSerial);

K_factor = 6; % Rician K-factor
ricianChan = comm.RicianChannel('SampleRate', 1e6, 'KFactor', K_factor, 'PathDelays', [0 2e-6], 'AveragePathGains', [0 -3]);
rxSerial_Rician = ricianChan(txSerial);

% Reshape back into OFDM symbol structure
rxOFDM_AWGN = reshape(rxSerial_AWGN, N+CP, numSymbols);
rxOFDM_Rayleigh = reshape(rxSerial_Rayleigh, N+CP, numSymbols);
rxOFDM_Rician = reshape(rxSerial_Rician, N+CP, numSymbols);

% Remove Cyclic Prefix
rxOFDM_AWGN = rxOFDM_AWGN(CP+1:end, :);
rxOFDM_Rayleigh = rxOFDM_Rayleigh(CP+1:end, :);
rxOFDM_Rician = rxOFDM_Rician(CP+1:end, :);

% Apply FFT
rxFFT_AWGN = fft(rxOFDM_AWGN, N);
rxFFT_Rayleigh = fft(rxOFDM_Rayleigh, N);
rxFFT_Rician = fft(rxOFDM_Rician, N);

% Convert back to serial data
rxData_AWGN = reshape(rxFFT_AWGN, N*numSymbols, 1);
rxData_Rayleigh = reshape(rxFFT_Rayleigh, N*numSymbols, 1);
rxData_Rician = reshape(rxFFT_Rician, N*numSymbols, 1);

% Demodulation
demodData_AWGN = pskdemod(rxData_AWGN, M, pi/4);
demodData_Rayleigh = pskdemod(rxData_Rayleigh, M, pi/4);
demodData_Rician = pskdemod(rxData_Rician, M, pi/4);

BER_AWGN = sum(data ~= demodData_AWGN) / length(data);
BER_Rayleigh = sum(data ~= demodData_Rayleigh) / length(data);
BER_Rician = sum(data ~= demodData_Rician) / length(data);

disp(['BER for AWGN Channel: ', num2str(BER_AWGN)]);
disp(['BER for Rayleigh Channel: ', num2str(BER_Rayleigh)]);
disp(['BER for Rician Channel: ', num2str(BER_Rician)]);

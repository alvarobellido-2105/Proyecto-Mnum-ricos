A.2. Script de MATLAB (adquisición, tabulación e interpolación)
Script que se conecta al puerto serial del Arduino, recibe las 21 muestras, construye el vector de tiempo, tabula los datos, realiza la interpolación por splines cúbicos, derivación numérica e integración numérica  y grafica los resultados.
clear; clc; close all;
puerto = "COM3"; 
baud = 9600;     
Fs = 4;          
limiteMuestras = 21;

arduino = serialport(puerto, baud);
flush(arduino);

fprintf('Conectado. Adquiriendo las %d muestras en tiempo real...\n', limiteMuestras);

muestras = zeros(1, limiteMuestras);
contador = 1;
while contador <= limiteMuestras
    linea = strtrim(readline(arduino)); 
    if ~isempty(linea)
        valor = str2double(linea);
        if ~isnan(valor)
            muestras(contador) = valor;
            fprintf('Muestra %d/%d recibida: %.4f V\n', contador, limiteMuestras, valor);
            contador = contador + 1;
        end
    end
end


clear arduino; 
fprintf('Adquisición finalizada con éxito. Procesando datos...\n');

N = length(muestras);
t = (0:N-1)/Fs;
dt = 1/Fs;      

tablaDatos = table(t', muestras', 'VariableNames', {'Tiempo_s', 'Voltaje_V'});
disp('=== TABLA DE DATOS EXPERIMENTALES ===');
disp(tablaDatos);

t_interp = linspace(min(t), max(t), N*20);

muestras_interp = spline(t, muestras, t_interp);


V_estimado = spline(t, muestras, 1.10);
fprintf('=========================================================\n');
fprintf('INTERPOLACIÓN POR SPLINES CÚBICOS:\n');
fprintf('Voltaje estimado en t = 1.10 s: %.4f V\n', V_estimado);

dV_dt = zeros(1, N);
dV_dt(1) = (muestras(2) - muestras(1)) / dt;

for i = 2:N-1
    dV_dt(i) = (muestras(i+1) - muestras(i-1)) / (2 * dt);
end

dV_dt(N) = (muestras(N) - muestras(N-1)) / dt;
dV_dt_interp = spline(t, dV_dt, t_interp);
V_integral_acum = zeros(1, N);

for i = 2:N
    area_trapecio = (dt / 2) * (muestras(i-1) + muestras(i));
    V_integral_acum(i) = V_integral_acum(i-1) + area_trapecio;
end


integral_total = V_integral_acum(end);
fprintf('\nINTEGRACIÓN NUMÉRICA:\n');
fprintf('El área total bajo la curva (0 a 5s) es: %.5f V*s\n', integral_total);
fprintf('=========================================================\n');

V_integral_acum_interp = spline(t, V_integral_acum, t_interp);

// GRÁFICA M1

figure('Name', 'Figura 4: Señal muestreada', 'NumberTitle', 'off');
stem(t, muestras, 'filled', 'LineWidth', 1.5, 'MarkerSize', 5);
grid on;
xlim([0 5]);
ylim([0 1.2]); 
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title('Señal muestreada');

figure('Name', 'Figura 5: Señal muestreada vs Interpolación', 'NumberTitle', 'off');
plot(t_interp, muestras_interp, 'b', 'LineWidth', 2); hold on;
stem(t, muestras, 'r', 'filled', 'LineWidth', 1.5, 'MarkerSize', 5);
plot(1.10, V_estimado, 'kp', 'MarkerFaceColor', 'k', 'MarkerSize', 10); 
grid on;
xlim([0 5]);
ylim([0 1.2]);
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title('Señal muestreada vs Interpolación por Splines Cúbicos');
legend('Interpolación por spline', 'Muestras', 'Estimación t=1.10s', 'Location', 'best');

// GRÁFICA 2

figure('Name', 'Derivada Numérica por Spline', 'NumberTitle', 'off');
plot(t_interp, dV_dt_interp, 'b', 'LineWidth', 2);
grid on;
xlim([0 5]);
xlabel('Tiempo (s)');
ylabel('Tasa de cambio (V/s)');
title('Derivada Numérica Continua mediante Splines Cúbicos');

figure('Name', 'Muestras de la Derivada Numérica', 'NumberTitle', 'off');
stem(t, dV_dt, 'filled', 'LineWidth', 1.5, 'MarkerSize', 5);
grid on;
xlim([0 5]);
xlabel('Tiempo (s)');
ylabel('Tasa de cambio (V/s)');
title('Muestras Calculadas de la Derivada Numérica (dV/dt)');

// GRÁFICA 3

figure('Name', 'Integral Numérica por Spline', 'NumberTitle', 'off');
plot(t_interp, V_integral_acum_interp, 'b', 'LineWidth', 2);
grid on;
xlim([0 5]);
xlabel('Tiempo (s)');
ylabel('Voltaje Integrado (V·s)');
title('Integral Numérica Acumulativa mediante Splines Cúbicos');


figure('Name', 'Área bajo la curva por Trapecios', 'NumberTitle', 'off');
area(t, V_integral_acum, 'FaceColor', [0.8 0.95 0.8], 'EdgeColor', [0 0.5 0], 'LineWidth', 1.5);
grid on;
xlim([0 5]);
xlabel('Tiempo (s)');
ylabel('Voltaje Integrado (V·s)');
title('Área bajo la curva (Regla del Trapecio Compuesta)');


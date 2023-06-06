# Project

clc
clear all
close all

% データの取得
f = fred;
startdate = '01/01/1994';
enddate = '07/01/2020';

japan_GDP = fetch(f, 'JPNRGDPEXP', startdate, enddate)
uk_GDP = fetch(f, 'CLVMNACSCAB1GQUK', startdate, enddate)
year = japan_GDP.Data(:, 1);
GDP_japan = japan_GDP.Data(:,2);
GDP_uk = uk_GDP.Data(:,2);

n = size(GDP_japan, 1);

% Hodrick-Prescott filter
lam = 1600;
A = zeros(n,n);

% unusual rows
A(1,1)= lam+1; A(1,2)= -2*lam; A(1,3)= lam;
A(2,1)= -2*lam; A(2,2)= 5*lam+1; A(2,3)= -4*lam; A(2,4)= lam;

A(n-1,n)= -2*lam; A(n-1,n-1)= 5*lam+1; A(n-1,n-2)= -4*lam; A(n-1,n-3)= lam;
A(n,n)= lam+1; A(n,n-1)= -2*lam; A(n,n-2)= lam;

% generic rows
for i=3:n-2
    A(i,i-2) = lam; A(i,i-1) = -4*lam; A(i,i) = 6*lam+1;
    A(i,i+1) = -4*lam; A(i,i+2) = lam;
end



tauGDP_japan = A\GDP_japan;
tauGDP_uk = A\GDP_uk;



% detrended GDP
GDP_japan_detrended = GDP_japan - tauGDP_japan;
GDP_uk_detrended = GDP_uk - tauGDP_uk;

% plot detrended GDP
figure
hold on
plot(year, GDP_japan_detrended, 'b')
plot(year, GDP_uk_detrended, 'r')
title(['Detrended Real GDP Comparison: Japan vs UK'])
xlabel('Year')
ylabel('Detrended GDP')
legend('Japan', ['UK'])
grid on

datetick('x', 'yyyy')


%[trend, cycle] = hpfilter(log(y), 1600);
[GDP_japan, trend] = qmacro_hpfilter(log(GDP_japan), 1600);
[GDP_uk, trend] = qmacro_hpfilter(log(GDP_uk), 1600);


% compute sd(y) (from detrended series)
ysd_japan = std(GDP_japan)*100;
ysd_uk = std(GDP_uk)*100;
corryc = corrcoef(GDP_japan_detrended(1:n),GDP_uk_detrended(1:n)); corryc = corryc(1,2);


disp(['Percent standard deviation of detrended log real GDP of Japan: ', num2str(ysd_japan),'.']); disp(' ')
disp(['Percent standard deviation of detrended log real GDP of UK: ', num2str(ysd_uk),'.']); disp(' ')
disp(['Contemporaneous correlation between GDP_japan_detrended and GDP_uk_detrended: ', num2str(corryc),'.']);


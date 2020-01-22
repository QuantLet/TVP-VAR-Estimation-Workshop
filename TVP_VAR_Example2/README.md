[<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/banner.png" width="888" alt="Visit QuantNet">](http://quantlet.de/)

## [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **TVP_VAR_Example2** [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/)

```yaml

Name of QuantLet : TVP_VAR_Example2

Published in : 'Haindorf Seminar 2020'

Description : 'S&P500 Daily Returns Rt = 100 × ln(Pt/Pt?1) from January 2, 1990 to December 31, 2018'

Keywords : 'S&P500, daily Returns, apple'

See also : 'TVP_VAR_Example1, TVP_VAR_Example3, TVP_VAR_Example4'

Author : 'Mike Ellington'

Submitted : January 22 2020
```

![Picture1](SP500_Daily_Returns.jpg)

### MATLAB Code
```matlab

% This file estimates stochastic volatility model on daily SP500 returns
% from 1990-01-01 -- 2018-12-31

% Model is as in slides r_{t} = \epsilon_{t}\sqrt(\exp(\ln(h_t)))
%                       \ln(h_t) = \ln(h_t-1) + \nu_{t}

clear; clc;
seed= 19122019;
rng(seed);
%%
addpath('../src');
addpath('..');

y=xlsread('Example_2_data','Sheet1','C3:C7308');
Ttrain=500; % training sample
datevec=xlsread('Example_2_data','Sheet1','A3:A7308');

datevec1=datetime(datevec,'ConvertFrom','yyyymmdd');
datevec2=datenum(datevec1);
figure(1)
plot(datevec2,y,'b-','LineWidth',2)
ylabel('%')
xlabel('Time')
legend('SP500 Daily Returns R_t=100 x LN(P_t/P_{t-1})','Location','SouthOutside')
axis tight
matlab2tikz('SP500_daily.tex')
%%

%
% independence Metropolis Hastings Algorithm for Stochastic Volatility
% Model Jacquer et al (2004).

% STEP 1: Priors for g~iG(V0,T0) and the initial conditions for the stochastic
% volatility
V0=0.01; % prior scale
T0=1; % prior degrees of freedom
mu=log(std(y(1:Ttrain))^2);
sigma=10;

% remove training sample from estimation sample
y=y(Ttrain+1:end,:); T=length(y);

% STEP 2: Starting Values for Stochastic Volatility
hlast=diff(y).^2;
hlast=[hlast(1:2); hlast]+0.001; % add small number to ensure no zero value

g=1;
Nsim=30000; burn=25000;
svol=zeros(T+1,Nsim-burn);
tic;
for kk=1:Nsim
% STEP 3: Date by date MH algorithm to draw SVOL
hnew=zeros(T+1,1);

i=1;
% time period 0 (initial condition)

hlead=hlast(i+1); 
ss=sigma*g/(g+sigma); % variance
mu1=ss*(mu/sigma + log(hlead)/g); % mean
% draw from lognormal distribution using mu and ss
h = exp(mu+(ss^.5)*randn(1,1));
hnew(i)=h;

% time period 1:t-1
for i=2:T
    hlead=hlast(i+1);
    hlag=hnew(i-1);
    yt=y(i-1);
   
%mean and variance of the proposal log normal density
mu = (log(hlead)+log(hlag))/2;  
ss = g/2;

%candidate draw from lognormal
htrial = exp(mu + (ss^.5)*randn(1,1));

%acceptance probability in logs
lp1 = -0.5*log(htrial) - (yt^2)/(2*htrial);  %numerator
lp0 = -0.5*log(hlast(i)) - (yt^2)/(2*hlast(i));   %denominator
accept = min([1;exp(lp1 - lp0)]);  %ensure accept<=1

u = rand(1,1);
if u <= accept;
   h = htrial;
else
   h = hlast(i);
end
hnew(i)=h;
end

%time period T
i=T+1;
yt=y(i-1);
hlag=hnew(i-1);
%mean and variance of the proposal density
mu = log(hlag);   % only have ht-1
ss = g;
%candidate draw from lognormal
htrial = exp(mu + (ss^.5)*randn(1,1));

%acceptance probability
lp1 = -0.5*log(htrial) - (yt^2)/(2*htrial);
lp0 = -0.5*log(hlast(i)) - (yt^2)/(2*hlast(i));
accept = min([1;exp(lp1 - lp0)]);  %ensure accept<=1


u = rand(1,1);
if u <= accept;
   h = htrial;
else
   h = hlast(i);
end
hnew(i)=h;

% STEP 4: Draw g from iG distribution
errors=diff(log(hnew));
g=iG(T0,V0,errors);  %draw from the inverse Gamma distribution


% STEP 5: update vale of h
hlast=hnew;
%save
if kk>burn
kk
svol(:,kk-burn)=hlast;
end
end
toc;
%%
SVQ=quantile(svol,[0.025, 0.5, 0.975],2);

figure(2)
plot(datevec2(Ttrain:end,:),SVQ(:,2),'k-','LineWidth',2)
hold on,
plot(datevec2(Ttrain:end),SVQ(:,1),'r--','LineWidth',1.2)
plot(datevec2(Ttrain:end),SVQ(:,3),'r--','LineWidth',1.2)
legend('Posterior median','95% coverage','Location','SouthOutside')
axis tight
matlab2tikz('SVOL.tex')

```

automatically created on 2020-01-22
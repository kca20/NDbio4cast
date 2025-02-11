022 ND BIOS Biological Forecasting Class Repository Zhuoran Yu email: zyu3@nd.edu

Kayla Anderson email: kander42@nd.edu

 main
Stacy Mowry email: smowry@nd.edu

Nick Andrysiak email: nandrysi@nd.edu

Stacy Mowry
email: smowry@nd.edu

Nick Andrysiak
email: nandrysi@nd.edu

## Download and visualize data

##load libraries
#install.packages("neonUtilities")
library(neonUtilities)
library(lubridate)


#Load Target Data
download_targets <- function(){
  readr::read_csv("https://data.ecoforecast.org/neon4cast-targets/aquatics/aquatics-targets.csv.gz", guess_max = 1e6)
}

target<-download_targets()

##subset target data
target<-subset(target, target$site_id == "PRPO")
target<-subset(target, target$variable == "chla")

##plot target data over time
plot(target$datetime,target$observation,type="p",xlab="Date",ylab = "Chl-A")

##Start and end date of data availability (2018-2022)
summary(target$datetime)

##resolution of data = DAILY
table(target$datetime)
target.date<-format(as.POSIXct( target$datetime,format='%m/%d/%Y'),format='%m/%d/%Y')

target.data<-cbind(target.date,target$observation)
colnames(target.data)<-c("date","target")



##% NAs
na.perc<-sum(is.na(target$observation))/nrow(target)

## pull predictor variables 
## dissolved o2 is available monthly (2016-2022)

  dissolved.gasses <- loadByProduct(dpID="DP1.20097.001", 
                                    site=c("PRPO"),
                                    check.size = F)
  
  ## extract date and dissolved 02 data
  time.gas<-dissolved.gasses$sdg_fieldSuperParent$collectDate
  time.gas<-lubridate::as_datetime(time.gas)
  
  ##format date to merge with target data
  time.gas.format<-format(as.POSIXct( time.gas,format='%m/%d/%Y %H:%M:%S'),format='%m/%d/%Y')
  dissolved.02<-dissolved.gasses$sdg_fieldSuperParent$dissolvedOxygen
  dissolved.02.time<-cbind(time.gas.format,dissolved.02)
  
  dissolved.O2.data<-cbind(time.gas.format,dissolved.02)
  colnames(dissolved.O2.data)<-c("date","observation")
  
  
  ##plot time series
  plot(time.gas,dissolved.02, type="p",xlab = "Dissolved O2", ylab= "time")
  
##data summaries 
summary(time.gas)
table(time.gas)

  



##Biomass 2014-2021, every few months

  biomass <- loadByProduct(dpID="DP1.20163.001", 
                           site=c("PRPO"),
                           check.size = F)
  
  ## extract date and biomass data
  time.biomass<-biomass$alg_fieldData$collectDate
  time.biomass<-lubridate::as_datetime(time.biomass)
  
  ##format date to merge with target data
  time.biomass.format<-format(as.POSIXct( time.biomass,format='%m/%d/%Y %H:%M:%S'),format='%m/%d/%Y')
 

  
  phyto<-biomass$alg_fieldData$phytoDepth1
  phyto.time<-cbind(time.biomass,phyto)
  phyto.time.format<-cbind(time.biomass.format,  phyto)
  colnames(phyto.time.format)<-c("date","observation")
  
  ##plot time series
  plot(time.biomass, phyto,type="p", xlab = "Time", ylab = "Biomass")
  
  #data summaries 
  print(summary(time.biomass))
  print(table(time.biomass))
  
  


##water temperature available from 2018-22
##data resolution every 30 minutes


  watertemp <- loadByProduct(dpID="DP1.20264.001", 
                             site=c("PRPO"),
                             check.size = F)
  
  ## extract date and water temp data 
  time.watertemp<-watertemp$TSD_30_min$startDateTime
  time.watertemp<-lubridate::as_datetime(time.watertemp)
  
  ##format date to merge with target data
  time.watertemp.format<-format(as.POSIXct( time.watertemp,format='%m/%d/%Y %H:%M:%S'),format='%m/%d/%Y')
  
temp_depth<-watertemp$TSD_30_min$tsdWaterTempMean
  
watertemp.time.format<-cbind(  time.watertemp.format, temp_depth)
watertemp.time<-cbind(time.watertemp,temp_depth)
colnames(watertemp.time.format)<-c("date","observation")
  
  ##plot time series
  plot(time.watertemp, temp_depth,type="p", xlab = "Time", ylab = "Water temp")
  
  #data summaries 
summary(time.watertemp)
table(time.watertemp)


##predictor variables vs target variable
##plot dissolved oxy vs target data
disO2.target<-merge(target.data, dissolved.O2.data, by="date", all=TRUE)
plot(disO2.target$observation,disO2.target$target,type="p",xlab = "Dissolved O2", ylab= "Chl-A")

##biomass vs chlorophyl
biomass.target<-merge(target.data, phyto.time.format, by="date", all=TRUE)
plot(biomass.target$observation,biomass.target$target,type="p",xlab ="Biomass",ylab ="Chl-A")

#water temp vs Chla
watertemp.target<-merge(target.data, watertemp.time.format, by="date", all=TRUE)
plot(watertemp.target$observation,watertemp.target$target,type="p",xlab ="Water temp",ylab ="Chl-A")


<<<<<<< HEAD
##Notes -- As a process model, we used a dynamical linear model where the future state of the system 
##is based on the current state. We also included water temperature as an additional linear predictor. 

## Our data model assumed random Gaussian noise. 

## we us ed gamma priors for our variance terms and weakly informative
## normal priors centered around 0 for our fixed effects

## Out trace plots indicated well mixed chains and the density plots
## of each of our paramters indicated convergence. 


## We noticed a strong correlation between beta.Intercept and beta.Temp, and are 
## not fully sure why that is there. Maybe just because there is always
## a correlation between an intercept and a slope in a linear model, however, 
## the correlation seems quite strong, so we are still thinking about this. 




library(neonUtilities) 
library(lubridate)
library(ggplot2)


#Load Target Data 
download_targets <- function(){ readr::read_csv("https://data.ecoforecast.org/neon4cast-targets/aquatics/aquatics-targets.csv.gz", guess_max = 1e6) }

target<-download_targets()
target$year <- year(target$datetime)
oxygen <- subset(target,site_id=="BARC"& variable=="oxygen"&year>"2019")
temperature <- subset(target,site_id=="BARC"& variable=="temperature"&year>"2019")
chla <- subset(target,site_id=="BARC"& variable=="chla"&year>"2019")

class(target$datetime)

oxygen.plot <- ggplot(oxygen, aes(x= datetime, y = observation)) + 
  geom_point()+ 
  theme_bw()+ theme(text = element_text(size=10)) + theme(legend.position = "none")+
  ylab("oxgen")+xlab("Year")+ labs(subtitle = "Oxygen")
oxygen.plot

temperature.plot <- ggplot(temperature, aes(x= datetime, y = observation)) + 
  geom_point()+ 
  theme_bw()+ theme(text = element_text(size=10)) + theme(legend.position = "none")+
  ylab("temperature")+xlab("Year")+ labs(subtitle = "Temperature")
temperature.plot

chla.plot <- ggplot(chla, aes(x= datetime, y = observation)) + 
  geom_point()+ 
  theme_bw()+ theme(text = element_text(size=10)) + theme(legend.position = "none")+
  ylab("chla")+xlab("Year")+ labs(subtitle = "chlorophyll a")
chla.plot

####only historical
time = as.Date(chla$datetime)
y = chla$observation
plot(time,y,type='l',ylab="Chla",lwd=2,log='y')

RandomWalk = "
model{
  
  #### Data Model
  for(t in 1:n){
    y[t] ~ dnorm(x[t],tau_obs)
  }
  
  #### Process Model
  for(t in 2:n){
    x[t]~dnorm(x[t-1],tau_add)
  }
  
  #### Priors
  x[1] ~ dnorm(x_ic,tau_ic)
  tau_obs ~ dgamma(a_obs,r_obs)
  tau_add ~ dgamma(a_add,r_add)
}
"
data <- list(y=log(y),n=length(y),      ## data
             x_ic=log(1000),tau_ic=100, ## initial condition prior
             a_obs=1,r_obs=1,           ## obs error prior
             a_add=1,r_add=1            ## process error prior
)
nchain = 3
init <- list()
for(i in 1:nchain){
  y.samp = sample(y,length(y),replace=TRUE)
  init[[i]] <- list(tau_add=1/var(diff(log(y.samp))),  ## initial guess on process precision
                    tau_obs=5/var(log(y.samp)))        ## initial guess on obs precision
}

library(rjags)
library(daymetr)
j.model   <- jags.model (file = textConnection(RandomWalk),
                         data = data,
                         inits = init,
                         n.chains = 3)
jags.out   <- coda.samples (model = j.model,
                            variable.names = c("tau_add","tau_obs"),
                            n.iter = 1000)
#plot(jags.out)
jags.out   <- coda.samples (model = j.model,
                            variable.names = c("x","tau_add","tau_obs"),
                            n.iter = 10000)

time.rng = c(1,length(time))       ## adjust to zoom in and out
out <- as.matrix(jags.out)         ## convert from coda to matrix  
x.cols <- grep("^x",colnames(out)) ## grab all columns that start with the letter x
ci <- apply(exp(out[,x.cols]),2,quantile,c(0.025,0.5,0.975)) ## model was fit on log scale

plot(time,ci[2,],type='n',ylim=range(y,na.rm=TRUE),ylab="Chla",log='y',xlim=time[time.rng],main="historical data")
## adjust x-axis label to be monthly if zoomed
if(diff(time.rng) < 100){ 
  axis.Date(1, at=seq(time[time.rng[1]],time[time.rng[2]],by='month'), format = "%Y-%m")
}
ecoforecastR::ciEnvelope(time,ci[1,],ci[3,],col=ecoforecastR::col.alpha("lightBlue",0.75))
points(time,y,pch="+",cex=0.5)

###add in water temperature
time = as.Date(chla$datetime)
y = chla$observation
plot(time,y,type='l',ylab="Chla",lwd=2,log='y')
Temp = temperature$observation
ef.out <- ecoforecastR::fit_dlm(model=list(obs="y",fixed="~ 1 + X + Temp"),data)
names(ef.out)

params <- window(ef.out$params,start=1000) ## remove burn-in
plot(params)

##trace plots look good -- chains look well mixed
summary(params)
cor(as.matrix(params))
## the very

pairs(as.matrix(params))

## confidence interval
out <- as.matrix(ef.out$predict)
ci <- apply(exp(out),2,quantile,c(0.025,0.5,0.975))
plot(time,ci[2,],type='n',ylim=range(y,na.rm=TRUE),ylab="Chla",log='y',xlim=time[time.rng],main="historical data and water temperature")
## adjust x-axis label to be monthly if zoomed
if(diff(time.rng) < 100){ 
  axis.Date(1, at=seq(time[time.rng[1]],time[time.rng[2]],by='month'), format = "%Y-%m")
}
ecoforecastR::ciEnvelope(time,ci[1,],ci[3,],col=ecoforecastR::col.alpha("lightBlue",0.75))
points(time,y,pch="+",cex=0.5)

## confidence interval
out <- as.matrix(ef.out$predict)
ci <- apply(exp(out),2,quantile,c(0.025,0.5,0.975))
plot(time,ci[2,],type='n',ylim=range(y,na.rm=TRUE),ylab="Flu Index",log='y',xlim=time[time.rng])
## adjust x-axis label to be monthly if zoomed
if(diff(time.rng) < 100){ 
  axis.Date(1, at=seq(time[time.rng[1]],time[time.rng[2]],by='month'), format = "%Y-%m")
}
ecoforecastR::ciEnvelope(time,ci[1,],ci[3,],col=ecoforecastR::col.alpha("lightBlue",0.75))
points(time,y,pch="+",cex=0.5)

## JAGS code for future reference
strsplit(ef.out$model,"\n",fixed = TRUE)[[1]]
=======
chla<-target$observation [target$site_id == "PRPO" & target$variable== "chla"]


#Downlodading chemical prop of surface water data
download_chem_prop<-function()
{
  chem_prop <- loadByProduct(dpID="DP1.20093.001", 
                                    site=c("PRPO"),
                                    check.size = F)
                                    
          #subset desired data
time<-chem_prop$swc_fieldSuperParent$collectDate
time<-lubridate::as_datetime(time)
oxygen<-chem_prop$swc_fieldSuperParent$dissolvedOxygen
temp<-chem_prop$swc_fieldSuperParent$waterTemp
chem_prop<-cbind(time,oxygen,temp)
  
  
  #return the dataframe
  chem_prop<<- as.data.frame(chem_prop)
  
  
  
}

oxygen<-chem_prop$oxygen
  plot (chla, chem_prop$oxygen)
  
  
  
>>>>>>> c97b86b3c6b26ebd89fc85fb4a11ea233f98098c




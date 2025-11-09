**📌 Project Title**
COVID-19 SQL Data Analysis Project

**🧠 Project Overview**
This project demonstrates the use of SQL to explore, clean, and analyze global COVID-19 data. It includes comparisons between infection rates, death tolls, vaccination efforts, and population-level metrics across countries and continents.

**📂 Dataset Summary**
Tables Used: CovidDeaths, CovidVaccinations
Source: Our World in Data

**🔍 Analysis Goals**
Analyze death rate relative to confirmed cases.
Calculate percentage of population infected by COVID-19.
Analyze vaccination trends by country and continent.
Create views and temporary tables for visualization in Tableau/Power BI.

**📊 Key SQL Skills Demonstrated**
Joins and CTEs
Partitioning / Window functions
Data cleaning and filtering
Creating views and temp tables for reporting

**##SQL SCRIPT**
select *
from [Portfolio Project].dbo.CovidDeaths$
where continent is not null
order by 3,4

--Select *
--From [Portfolio Project].dbo.CovidVaccinations$
--Order by 3,4

-- Lets start by selecting the data we need
select location, date,total_cases, total_deaths,new_cases,population
from [Portfolio Project].dbo.CovidDeaths$
order by 1,2
--Looking at Total cases vs total deaths

select location, date,total_cases, total_deaths,population,(total_deaths/total_cases)*100 AS DeathPercentage
from [Portfolio Project].dbo.CovidDeaths$
Where location like '%Nigeria%'
order by 1,2

--Lets look at the total cases vs the Population
--shows the percentage of people who contacted covid

select location, date,total_cases,population,(total_cases/population)*100 AS covidpercentage
from [Portfolio Project].dbo.CovidDeaths$
--where location like '%Nigeria%'
order by 1,2

--looking at countries with highest inflation rate compared to their population

select location, population,Max (total_cases) as highestinfectioncount,max((total_cases/population))*100 AS covidpercentage
from [Portfolio Project].dbo.CovidDeaths$
--where location like '%Nigeria%'
Group by location, population
order by 1,2

--Showing the countries with highest death count by population

Select location, max(cast (total_deaths as int)) as total_deathcount
from [Portfolio Project].dbo.CovidDeaths$
Where continent is not null
Group by location
Order by total_deathcount desc

-- Or checking out where continent is null
Select location, max(cast (total_deaths as int)) as total_deathcount
from [Portfolio Project].dbo.CovidDeaths$
Where continent is null
Group by location
Order by total_deathcount desc

--Lets break it down by Continent

Select continent, max(cast (total_deaths as int)) as total_deathcount
from [Portfolio Project].dbo.CovidDeaths$
Where continent is not null
Group by continent
Order by total_deathcount desc

--where continent is null by continent

Select continent, max(cast (total_deaths as int)) as total_deathcount
from [Portfolio Project].dbo.CovidDeaths$
Where continent is null
Group by continent
Order by total_deathcount desc

--Global numbers

select date, sum(new_cases) as total_cases, sum (cast(new_deaths as int)) as total_deaths, sum(cast(new_deaths as int))/sum(new_cases)*100 as Deathpercentage
from [Portfolio Project].dbo.CovidDeaths$
where continent is not null
Group by date
order by 1,2


---For the Vaccination Table (Lets try to join the tables)
select *
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
   --Looking at total populations Vs Vaccinations

select Dea.continent, Dea.location,Dea.date, Dea.population,Vac.new_vaccinations
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
where Dea.continent is not null
   Order by 1,2,3

   --Using the partition by Location

 select Dea.continent, Dea.location,Dea.date, Dea.population,Vac.new_vaccinations, sum(cast(Vac.new_vaccinations as int)) over (partition by Dea.Location  Order by Dea.location
 ,Dea. date) Total_Vac
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
where Dea.continent is not null
   Order by 2,3

   --Using a CTE Table

 With PopvsVac  (Continent, location, date, population, new_vaccinations, Total_Vac)
 as
(
select Dea.continent, Dea.location,Dea.date, Dea.population,Vac.new_vaccinations, sum(cast(Vac.new_vaccinations as int)) over (partition by Dea.Location  Order by Dea.location
 ,Dea. date)  Total_Vac
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
where Dea.continent is not null
 --Order by 2,3
)
Select *, (Total_Vac/population)*100  as Vac_Percentage
From PopvsVac


--Looking at the Temp Table

Drop table if exists #percentagepopulationvaccinated
Create table #percentagepopulationvaccinated
(Continent nvarchar(255),
Location nvarchar (255),
date datetime,
population numeric,
New_vaccinations numeric,
Total_Vac numeric
)
Insert into  #percentagepopulationvaccinated
select Dea.continent, Dea.location,Dea.date, Dea.population,Vac.new_vaccinations, sum(cast(Vac.new_vaccinations as int)) over (partition by Dea.Location  Order by Dea.location
 ,Dea. date)  Total_Vac
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
--where Dea.continent is not null
 --Order by 2,3
 Select *, (Total_Vac/population)*100  as Vac_Percentage
From #percentagepopulationvaccinated

--Creating View to store data for leta visualizations on Tableau
create view percentagepopulationvaccinated as
select Dea.continent, Dea.location,Dea.date, Dea.population,Vac.new_vaccinations, sum(cast(Vac.new_vaccinations as int)) over (partition by Dea.Location  Order by Dea.location
 ,Dea. date)  Total_Vac
from [Portfolio Project].dbo.CovidDeaths$  Dea
join [Portfolio Project].dbo.CovidVaccinations$  Vac
 on Dea.location = Vac.location
   and Dea.date = Vac.date
where Dea.continent is not null
 --Order by 2,3

 select*
 from #percentagepopulationvaccinated


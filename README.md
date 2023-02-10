# Covid_19_Death_Project

Select *
From Covid19DeathProject.dbo.Covid_Deaths
where continent is not null 
Order by 3,4

--Select *
--From Covid19DeathProject..Covid_Vaccinations
--order by 3,4

--We are working on Covid_Deaths dataset.

Select location, date, total_cases, new_cases, total_deaths, population
From Covid19DeathProject.dbo.Covid_Deaths
where continent is not null 
Order by 1,2

--Looking at Total cases vs Total Deaths
Select location, date, total_cases, total_deaths
From Covid19DeathProject.dbo.Covid_Deaths
where continent is not null 
Order by 1,2

--To get the %tage of Total cases vs Total Deaths
--Observe death rates increased to as much as 6%
Select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From Covid19DeathProject.dbo.Covid_Deaths
where continent is not null 
Order by 1,2

--To get the %tage of Total cases vs Total Deaths in United States
--Observe death rates increased to as much as 6%. By the end of 2020, we had a total of 20,219,867 total cases and 350,568 deaths.
Select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From Covid19DeathProject.dbo.Covid_Deaths
Where location like '%states%'
and continent is not null 
Order by 1,2

--Total cases vs population
--shows what percentage of population got covid comparing the 1% death rate to the 10% death rate and the number of people affected.
Select location, date, population, total_cases,  (total_cases/population)*100 as DeathPercentage
From Covid19DeathProject.dbo.Covid_Deaths
Where location like '%states%'
and continent is not null 
Order by 1,2


--Country that has the highest infection rate compared to the population
Observed, Cyprus has the highest population infection at 71.8% while United states was 30.3% and North Korea was 0.00000038%
Select location, population, MAX(total_cases) as HighestInfectionCount, Max(total_cases/population)*100 as PercentagePopulationInfected
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
Group by location, population
Order by PercentagePopulationInfected desc


--Countries with highest death count per population
Select location, MAX(cast(total_deaths as int)) as TotalDeathCount
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
Group by location
Order by TotalDeathCount desc


--To break down by Continent
Select continent, MAX(cast(total_deaths as int)) as TotalDeathCount
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
Group by continent
Order by TotalDeathCount desc

Select location, MAX(cast(total_deaths as int)) as TotalDeathCount
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is null 
Group by location
Order by TotalDeathCount desc

--Showing continents with the highest death count per poulation

Select continent, MAX(cast(total_deaths as int)) as TotalDeathCount
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
Group by continent
Order by TotalDeathCount desc

--Global numbers...we have to use aggreatate functions like SUM
Select date, SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
group by date
Order by 1,2

--Global number summed together
Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From Covid19DeathProject.dbo.Covid_Deaths
--Where location like '%states%'
where continent is not null 
--group by date
Order by 1,2


Select *
From Covid19DeathProject..Covid_Vaccinations



--We are working on Covid_Vaccinations dataset.
--Let join the 2 tables together

Select *
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date

-- Looking at Total Population vs Vaccinations
--dea is used for columns tht are similar on both tables
--observed that they start vaccination early in some states unlike others.
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2,3

--To add up vaccination report
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(Cast(vac.new_vaccinations as int)) OVER (Partition by dea.location)
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2,3

OR

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location)
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2,3




Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location order by dea.location, 
dea.date) as RollingPeopleVaccinated
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2,3

--To know the number of people vaccinated in a country
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, 
dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
order by 2,3

--Use CTE

with PopvsVac (Continent, Location, Date, Population, new_vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, 
dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
select * 
From PopvsVac

--TEMP TABLE
DROP Table if exists #PercentPopulationVaccinated
Create Table #percentpopulationvaccinated
(
Continent nvarchar (255),
Location nvarchar (255),
Date datetime,
population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location,
 dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null
--order by 2,3

Select*, (RollingPeopleVaccinated/population)*100
From #PercentPopulationVaccinated

--Creating view to store date for later visualization

Create view PercentPopulationVaccinated as 
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, 
dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Covid19DeathProject..Covid_Deaths dea
Join Covid19DeathProject..Covid_Vaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
--order by 2,3

Select *
From PercentPopulationVaccinated



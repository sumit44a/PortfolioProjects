Create Database potfolio_project
use database potfolio_project

create or replace table covid_deaths(
iso_code varchar(10),
continent varchar(100),
location varchar(225),
population varchar(100),
date string,
total_cases	varchar(100),	
new_cases varchar(100),	
new_cases_smoothed varchar(100),
total_deaths varchar(100),
new_deaths	varchar(100),
new_deaths_smoothed varchar(100),
total_cases_per_million	varchar(100),
new_cases_per_million	varchar(100),	
new_cases_smoothed_per_million varchar(100),
total_deaths_per_million	varchar(100),	
new_deaths_per_million	varchar(100),	
new_deaths_smoothed_per_million	varchar(100),	
reproduction_rate	varchar(100),	
icu_patients bigint,
icu_patients_per_million varchar(100),	
hosp_patients	bigint,
hosp_patients_per_million varchar(100),
weekly_icu_admissions	bigint,
weekly_icu_admissions_per_million varchar(100),	
weekly_hosp_admissions	bigint,
weekly_hosp_admissions_per_million varchar(100)
);

select * from covid_deaths

create or replace table covid_vaccination(
iso_code varchar(10),
continent varchar(100),
location varchar(225),
date	string,
total_tests varchar(100),
new_tests varchar(100),
total_tests_per_thousand varchar(100),
new_tests_per_thousand varchar(100),
new_tests_smoothed varchar(100),
new_tests_smoothed_per_thousand varchar(100),
positive_rate varchar(100),
tests_per_case	varchar(100),
tests_units	varchar(100),
total_vaccinations	varchar(100),
people_vaccinated varchar(100),
people_fully_vaccinated	varchar(100),
total_boosters varchar(100),
new_vaccinations	varchar(100),
new_vaccinations_smoothed	varchar(100),
total_vaccinations_per_hundred	varchar(100),
people_vaccinated_per_hundred varchar(100),
people_fully_vaccinated_per_hundred varchar(100),
total_boosters_per_hundred varchar(100),
new_vaccinations_smoothed_per_million varchar(100),
new_people_vaccinated_smoothed varchar(100),
new_people_vaccinated_smoothed_per_hundred	varchar(100), 
stringency_index varchar(100),
population_density	varchar(100),
median_age	varchar(100),
aged_65_older varchar(100),
aged_70_older	varchar(100),
gdp_per_capita varchar(100),
extreme_poverty	varchar(100),
cardiovasc_death_rate	varchar(100),
diabetes_prevalence varchar(100),
female_smokers varchar(100),
male_smokers varchar(100),
handwashing_facilities varchar(100),
hospital_beds_per_thousand varchar(100),
life_expectancy varchar(100),
human_development_index	varchar(100),
excess_mortality_cumulative_absolute varchar(100),
excess_mortality_cumulative	varchar(100),
excess_mortality varchar(100),
excess_mortality_cumulative_per_million varchar(100)
);

select * from covid_vaccination
order by 3,4

select * from covid_deaths
order by 3,4

select  location, date, total_cases, new_cases, total_deaths, population
from covid_deaths
order by 1,2

---looking at total case vs total deaths
---show likelihood of dying if you contact covid in your country
select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from covid_deaths
where location like '%India%'
order by 1,2

--Looking at Total_case vs Population
--show what percentage of people got covid

select location, date, population, total_cases, (total_cases/population)*100 as INFECTION_PERCENTAGE
from covid_deaths
where location like '%India%'
order by 1,2


--Looking for a country with highest infection rate compared to population

select location, date, population, total_cases, (total_cases/population)*100 as INFECTION_PERCENTAGE
from covid_deaths
where location like '%India%'
order by 1,2

--Looking at countries with Highest Infection Rate Compared to Population
select location, population, Max(total_cases) as HighestInfectionCount, Max(total_cases/population)*100 as PercentagePopulationInfection
from covid_deaths
group by location, population
order by PercentagePopulationInfection desc

--showing countries with highest Death Count per Population
select location, Max(cast(total_deaths as int)) as TotalDeathCount
from covid_deaths
where continent is not null and total_deaths is not null
group by location 
order by TotalDeathCount desc 

--Break things down by continent
--showing continents with the highest death count per population

select continent, Max(cast(total_deaths as int)) as TotalDeathCount
From covid_deaths
where continent is not null
group by continent
order by TotalDeathCount desc

--Global numbers

select sum(new_cases) as total_cases, sum(cast(new_deaths as int)) as total_deaths, 
sum(cast(new_deaths as int))/sum(new_cases)*100 as death_percentage
from covid_deaths
where continent is not null
order by 1,2

--Look at Total population vs Vaccination
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from covid_deaths dea
join covid_vaccination vac
     on dea.location = vac.location
     and dea.date = vac.date
where dea.continent is not null
order by 2,3


--USE CTE

with popvsvac(continent, location,date,population,new_vaccinations, RollingPeoplevaccinated)
as
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from covid_deaths dea
join covid_vaccination vac
     on dea.location = vac.location
     and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
select *, (RollingPeopleVaccinated/population) * 100 as RollingPeopleVaccinatedPercentage
from popvsvac


create or replace table percentagepopulationvaccinated(
continent nvarchar(255),
location nvarchar(255),
date string,
population numeric,
new_vaccination numeric,
RollingPeopleVaccinated numeric,
RollingPeopleVaccinatedPercentage decimal)

--create view

create view percentagepopulationvaccinated1 as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from covid_deaths dea
join covid_vaccination vac
     on dea.location = vac.location
     and dea.date = vac.date
where dea.continent is not null
--order by 2,3

select  
continent,
location,
date,
population,
new_vaccination,
RollingPeopleVaccinated
from percentagepopulationvaccinated limit 1000
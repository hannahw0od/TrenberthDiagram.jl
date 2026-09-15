# TrenberthDiagram.jl

`TrenberthDiagram.jl` provides tools for creating an interactive Trenberth-style energy budget diagram from [SpeedyWeather.jl](https://github.com/SpeedyWeather/SpeedyWeather.jl) simulations.

It works alongside `TrenberthCallbacks.jl` to collect model diagnostics and visualise the resulting energy fluxes.

## Installation

`TrenberthDiagram.jl` can be installed directly from GitHub.

Open the Julia package manager by pressing `]` in the Julia REPL, then run:

```julia
add https://github.com/hannahw0od/TrenberthDiagram.jl.git
```

Then load the required packages:

```julia
using TrenberthCallbacks
using TrenberthDiagram
using SpeedyWeather
using Dates
```

## Example

### 1. Create the model

```julia
spectral_grid = SpectralGrid()
model = PrimitiveWetModel(spectral_grid)
```

### 2. Add the required diagnostics

The Trenberth callback requires radiation and surface-flux diagnostics:

```julia
add!(model, SpeedyWeather.RadiationOutput()...)
add!(model, SpeedyWeather.SurfaceFluxesOutput()...)
```

### 3. Add the Trenberth callback

Create a callback that records the required diagnostics every 12 hours:

```julia
cb = TrenberthCallbacks.TrenberthCallback(
    schedule = Schedule(every = Hour(12))
)

add!(model.callbacks, :trenberth => cb)
```

### 4. Run the simulation

```julia
sim = initialize!(model)
run!(sim, period = Day(10))
```

### 5. Build the diagram data

After the simulation has finished, construct the observables used by the diagram:

```julia
obs = build_trenberth_observables(cb)
arrows = build_flux_arrow_observables(obs.current_point)
solar = get_solar_constant(model)
```

By default, `build_trenberth_observables` skips the initial/pre-run timestep.

### 6. Create the interactive diagram

```julia
fig, cleanup = plot_trenberth_diagram(obs, arrows, solar)
display(fig)
```

The resulting figure provides an interactive visualisation of the simulated energy budget.

## Complete Example

```julia
using TrenberthCallbacks
using TrenberthDiagram
using SpeedyWeather
using Dates

# Set up the model
spectral_grid = SpectralGrid()
model = PrimitiveWetModel(spectral_grid)

# Add diagnostics required by the Trenberth callback
add!(model, SpeedyWeather.RadiationOutput()...)
add!(model, SpeedyWeather.SurfaceFluxesOutput()...)

# Add the Trenberth callback
cb = TrenberthCallbacks.TrenberthCallback(
    schedule = Schedule(every = Hour(12))
)
add!(model.callbacks, :trenberth => cb)

# Run the simulation
sim = initialize!(model)
run!(sim, period = Day(10))

# Build observables for the diagram
obs = build_trenberth_observables(cb)
arrows = build_flux_arrow_observables(obs.current_point)
solar = get_solar_constant(model)

# Create and display the interactive diagram
fig, cleanup = plot_trenberth_diagram(obs, arrows, solar)
display(fig)
```

## Related Packages

- **TrenberthCallbacks.jl** — collects the diagnostics required to construct the Trenberth energy budget.
- **SpeedyWeather.jl** — provides the atmospheric model and simulation output used by the diagram.


[![Build Status](https://github.com/hannahw0od/TrenberthDiagram.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/hannahw0od/TrenberthDiagram.jl/actions/workflows/CI.yml?query=branch%3Amain)

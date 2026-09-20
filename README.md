!pip install openmm OpenMM-CUDA-12 MDAnalysis plotly pandas numpy
from openmm import *
from openmm.app import *
from openmm.unit import *
from sys import stdout



inpcrd=AmberInpcrdFile("pep4_last_frame.inpcrd")
prmtop=AmberPrmtopFile("5_4_bestpose_complex.prmtop")

#system=prmtop.createSystem(nonbondedMethod=NoCutoff, constraints=HBonds,    implicitSolvent=OBC2,    soluteDielectric=1.0,   solventDielectric=78.5)


system=prmtop.createSystem(nonbondedMethod=PME,nonbondedCutoff=1.0*nanometer, constraints=HBonds)
integrator=LangevinIntegrator(300*kelvin, 1/picosecond, 0.002*picoseconds)
platform = Platform.getPlatformByName('CUDA')
platformProperties = {
    'Precision': 'mixed'
    #,'DeviceIndex': '0,1'  # Tells OpenMM to use both T4 GPUs!
}
simulation=Simulation(prmtop.topology, system, integrator,platform,platformProperties)

# Line 15: Sets the atom coordinates
simulation.context.setPositions(inpcrd.positions)



#simulation.minimizeEnergy(tolerance=0.1*kilojoule/mole/nanometer, maxIterations=50000)
print("Minimization done", flush=True)

# check energy is finite before continuing
state = simulation.context.getState(getEnergy=True)
print(f"Post-minimization PE: {state.getPotentialEnergy()}", flush=True)



simulation.reporters.append(DCDReporter("Ppe_4_micro_second_md_traj_200_500.dcd",100000))
simulation.reporters.append(StateDataReporter(stdout, 1000, step=True, time=True, potentialEnergy=True, temperature=True, speed=True))
simulation.context.setVelocitiesToTemperature(300*kelvin)



simulation.step(150000000)
print("Production done", flush=True)

from galaxy_ie_helpers import put

# Replace with your actual DCD file name
put('Ppe_4_micro_second_md_traj_200_500.dcd')

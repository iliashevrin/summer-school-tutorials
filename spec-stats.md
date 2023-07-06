### Specification Statistics Menu Item

We would like to add a right click menu item in the GUI of Spectra that prints to the console some statistics on the specification, such as the number of assumtions/guarantees by kind, and the node count of the BDD represantation of the initial and transition formulas of the system and the environment.

1. First, create a separate git branch from master, by running `git checkout -b [YOUR_NAME]-tutorial-spec-stats origin/master`.
2. Create a Java class called `StatsJob` under tau.smlab.syntech.ui.jobs package in tau.smlab.syntech.ui project.
3. Paste the following content for the class:
   ```java
   public class StatsJob extends SyntechJob {

    	@Override
    	protected void doWork() {
    
    		clearMarkers();
    
    		printToConsole("");
    		
    		printToConsole("Environment variables count: " + model.getEnv().getAllFields().size());
    		printToConsole("System variables count: " + model.getSys().getAllFields().size());
    		printToConsole("Of which auxiliary variables: " + model.getSys().getAuxFields().size());
    		printToConsole("");
    		
    		printToConsole("Environment initial constraints count: " + model.getEnvBehaviorInfo().stream().filter(b -> b.isInitial()).count());
    		printToConsole("Environment safety constraints count: " + model.getEnvBehaviorInfo().stream().filter(b -> b.isSafety()).count());
    		printToConsole("Environment justice constraints count: " + model.getEnvBehaviorInfo().stream().filter(b -> b.isJustice()).count());
    		printToConsole("");
    		
    		printToConsole("System initial constraints count: " + model.getSysBehaviorInfo().stream().filter(b -> b.isInitial()).count());
    		printToConsole("System safety constraints count: " + model.getSysBehaviorInfo().stream().filter(b -> b.isSafety()).count());
    		printToConsole("System justice constraints count: " + model.getSysBehaviorInfo().stream().filter(b -> b.isJustice()).count());
    		printToConsole("");
    		
    		printToConsole("Auxiliary initial constraints count: " + model.getAuxBehaviorInfo().stream().filter(b -> b.isInitial()).count());
    		printToConsole("Auxiliary safety constraints count: " + model.getAuxBehaviorInfo().stream().filter(b -> b.isSafety()).count());
    		printToConsole("");
    		
    		printToConsole("Environment initial BDD node count: " + model.getEnv().initial().nodeCount());
    		printToConsole("Environment transitions BDD node count: " + model.getEnv().trans().nodeCount());
    		printToConsole("");
    		
    		printToConsole("System initial BDD node count: " + model.getSys().initial().nodeCount());
    		printToConsole("System transitions BDD node count: " + model.getSys().trans().nodeCount());
    		printToConsole("");
    		
    		model.free();
     }
   }
   ```
4. Open the `SynthesisAction` class. This class contains a switch case for all the different jobs that you can execute within the GUI of Spectra. Add a new case as follows:
   ```java
   		case "tau.smlab.syntech.statsAction":
			  job = new StatsJob();
			  job.setTrace(TraceInfo.ALL);
			  break;
   ```
5. Open the `plugin.xml` configuration file. This is an XML file that contains meta data on all the jobs, and how they are displayed inside a right click menu. Add the following XML content:
   ```xml
      <action
        id="tau.smlab.syntech.statsAction" 
        label="Specification Statistics"
        icon="icons/analyze-wheel.gif"        
        menubarPath="tau.smlab.syntech.ui.syntechmenu/group1" 
        class="tau.smlab.syntech.ui.action.SynthesisAction"> 
      </action>
   ```
   You can add this XML in the editor view section, in the explorer view section, or both.
6. Compile everything and open the development Eclipse application.
7. Right click on a specification file, choose "Specification Statistics" and look at the output in the console.

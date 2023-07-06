### Support for If-Then-Else Language Construct

We would like to enrich the Spectra specification language with a construct of if-then-else that takes three Boolean formulas a,b,c as parameters, and defines a formula that semantically represents "if a then b, else c".

1. Create a separate git branch in `spectra-synt`, tracking master, by running `git checkout -b [YOUR_NAME]-tutorial-if-then-else origin/master` in the `spectra-synt` directory.
2. Create a separate git branch in `spectra-lang`, tracking master, by running `git checkout -b [YOUR_NAME]-tutorial-if-then-else origin/master` in the `spectra-lang` directory.
3. Open the `Spectra.xtext` file, which defines the grammar of the Spectra specification language, and add the line with `ite` to the `TemporalPrimaryExpression` rule:
   ```abnf
   TemporalPrimaryExpr returns TemporalExpression:
	Constant
	| '(' QuantifierExpr ')'
	| {TemporalPrimaryExpr}
	 (predPatt=[PredicateOrPatternReferrable] ('(' predPattParams+=TemporalInExpr (',' predPattParams+=TemporalInExpr)* ')' | '()')
   
	 | 'ite' '(' ifPart=TemporalInExpr ',' thenPart=TemporalInExpr ',' elsePart=TemporalInExpr ')' // THIS LINE IS NEW
   
	 | operator=('-'|'!') tpe=TemporalPrimaryExpr
	 | pointer=[Referrable]('[' index+=TemporalInExpr ']')* 
	 | operator='next' '(' temporalExpression=TemporalInExpr ')'
	 | operator='regexp' '(' (regexp=RegExp | regexpPointer=[DefineRegExpDecl]) ')'
	 | pointer=[Referrable] operator='.all'
	 | pointer=[Referrable] operator='.any'
	 | pointer=[Referrable] operator='.prod'
	 | pointer=[Referrable] operator='.sum'
	 | pointer=[Referrable] operator='.min'
	 | pointer=[Referrable] operator='.max');
   ```
4. Regenerate the Xtext artifacts (the abstract syntax tree objects) by right clicking on the `Spectra.xtext` file, "Run As", and "Generate Xtext Artifacts".
5. Open the `TemporalPrimaryExpression` class, which is part of the auto-generated artifacts, and ensure that you see the new getters and setters for `ifPart`, `thenPart`, and `elsePart`.
6. Create a Java class called `IfThenElseInstance` under tau.smlab.syntech.gameinput.spec package in tau.smlab.syntech.gameinput project.
7. Paste the following content into the class:
   ```java
   public class IfThenElseInstance implements Spec {
  
  	  private static final long serialVersionUID = 165285782221463029L;
  	
    	private Spec ifPart;
    	private Spec thenPart;
    	private Spec elsePart;
    
    	public IfThenElseInstance(Spec ifPart, Spec thenPart, Spec elsePart)
    	{
    		this.ifPart = ifPart;
    		this.thenPart = thenPart;
    		this.elsePart = elsePart;
    	}
    
    	public String toString()
    	{
    		return "<IfThenElseInstance>";
    	}
    
    	public Spec getIfPart() {
    		return ifPart;
    	}
    
    	public void setIfPart(Spec ifPart) {
    		this.ifPart = ifPart;
    	}
    
    	public Spec getThenPart() {
    		return thenPart;
    	}
    
    	public void setThenPart(Spec thenPart) {
    		this.thenPart = thenPart;
    	}
    
    	public Spec getElsePart() {
    		return elsePart;
    	}
    
    	public void setElsePart(Spec elsePart) {
    		this.elsePart = elsePart;
    	}
    
    	@Override
    	public boolean isPastLTLSpec() {
    		// TODO Auto-generated method stub
    		return false;
    	}
    
    	@Override
    	public boolean isPropSpec() {
    		// TODO Auto-generated method stub
    		return false;
    	}
    
    	@Override
    	public boolean hasTemporalOperators() {
    		// TODO Auto-generated method stub
    		return false;
    	}
    	
    	@Override
    	public Spec clone() throws CloneNotSupportedException {
    		return new IfThenElseInstance(ifPart.clone(), thenPart.clone(), elsePart.clone());
    	}
   }
   ```
8. Open the `SpectraASTToSpecGenerator` class. This class contains code that creates a `SpecWrapper` object from the auto-generated artifacts. This class wraps a `Spec` object, which is later translated to a BDD. Add the following code to the `getConstraintSpec` method in line 399 (note that there are many overloads of this method in the class):
   ```java
   if (temporalPrimaryExpr.getIfPart() != null) {
			
			return new SpecWrapper(new IfThenElseInstance(
				getConstraintSpec(temporalPrimaryExpr.getIfPart(), entitiesMapper, tracer, predicateParamsList, patternVarsAndParams, constraintParamsValues).getSpec(), 
				getConstraintSpec(temporalPrimaryExpr.getThenPart(), entitiesMapper, tracer, predicateParamsList, patternVarsAndParams, constraintParamsValues).getSpec(), 
				getConstraintSpec(temporalPrimaryExpr.getElsePart(), entitiesMapper, tracer, predicateParamsList, patternVarsAndParams, constraintParamsValues).getSpec()));
		}
   ```
9. Create a Java class called `IfThenElseInstanceTranslator` under tau.smlab.syntech.gameinputtrans.translator package in tau.smlab.syntech.gameinputtrans project. This class translates the `IfThenElseInstance` to a simple `Spec` instance without any additional language constructs, just plain Boolean operators.
10. Paste the following content into the class:
    ```java
    public class IfThenElseInstanceTranslator implements Translator {

    	@Override
    	public void translate(GameInput input) {
    
    		for (Constraint c : input.getSys().getConstraints()) {
    			c.setSpec(replaceIfThenElseInstances(c.getSpec()));
    		}
    
    		for (Constraint c : input.getEnv().getConstraints()) {
    			c.setSpec(replaceIfThenElseInstances(c.getSpec()));
    		}
    
    		for (Constraint c : input.getAux().getConstraints()) {
    			c.setSpec(replaceIfThenElseInstances(c.getSpec()));
    		}
    	}
    
    	private Spec replaceIfThenElseInstances(Spec spec) {
    		if (spec instanceof IfThenElseInstance) {
    			IfThenElseInstance pi = (IfThenElseInstance) spec;
    
    			spec = translateIfThenElse(pi);
    			
    			spec = replaceIfThenElseInstances(spec);
    			
    		} else if (spec instanceof SpecExp) {
    			SpecExp se = (SpecExp) spec;
    			for (int i = 0; i < se.getChildren().length; i++) {
    				se.getChildren()[i] = replaceIfThenElseInstances(se.getChildren()[i]);
    			}
    		}
    		return spec;
    	}
    	
    	private Spec translateIfThenElse(IfThenElseInstance ifThenElse) {
    		
    		Spec ifThen = new SpecExp(Operator.IMPLIES, ifThenElse.getIfPart(), ifThenElse.getThenPart());
    		Spec elseThen = new SpecExp(Operator.IMPLIES, new SpecExp(Operator.NOT, ifThenElse.getIfPart()), ifThenElse.getElsePart());
    		return new SpecExp(Operator.AND, ifThen, elseThen);
    	}
    }
    ```
11. Open the `DefaultTranslators` class. In this class, all the translators register themselves to the translator list. Add the following line in line 43 (the order in which the translators execute matters):
    ```java
    ts.add(new IfThenElseInstanceTranslator());
    ```
12. Compile everything and open the development Eclipse application.
13. Add the following Spectra specification to the project:
    ```
    module IfThenElseTest

    env boolean carA; 
    env boolean carB; // cars sensors on lanes A and B
    
    sys boolean greenA; 
    sys boolean greenB; //  green lights on lanes A and B
    
    // assumptions
    asm ini !carA & !carB; // initially no cars
    asm alwEv carA; 
    asm alwEv carB; // infinitely many cars on each lane
    
    // Equivalent to "gar alw (carA -> greenA) & (!carA -> greenB);"
    gar alw ite(carA, greenA, greenB);
    
    gar alw !(greenA & greenB); // never both green
    
    gar alwEv carA & greenA; // always eventually car and green on lane A
    gar alwEv carB & greenB; // always eventually car and green on lane B
    ```
14. Check realizability and notice that this specification is unrealizable. Can you explain why?
15. Comment out the line with the `ite` formula. What is the realizability status now?

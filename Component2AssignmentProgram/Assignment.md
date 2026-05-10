Refer to the assignment spec on myBeckett


There is useful information in the assignment spec directory on myBeckett, including help videos

package Myrailway;

import java.util.Scanner;

public class MainClass {

    public static void main(String[] args) {

        MyRailway railway = new MyRailway();

        // Create locomotive
        NewLoco loco = new NewLoco(railway, railway.getWorld(), 4, 3);
        
        Scanner scan = new Scanner(System.in);

        System.out.println("===== RAILWAY CONTROL SYSTEM =====");
        System.out.println("Commands:");
        System.out.println("start");
        System.out.println("stop");
        System.out.println("about");
        System.out.println("green  -> Train moves");
        System.out.println("red    -> Train stops");
        System.out.println("danger -> Emergency stop");
        System.out.println("emergency -> Global System Override");
        System.out.println("exit");

        while (scan.hasNextLine()) {

            String input = scan.nextLine();

            if (input.equalsIgnoreCase("about")) {
                railway.about();
            }

            else if (input.equalsIgnoreCase("green")) {
                loco.startTrain();
            }

            else if (input.equalsIgnoreCase("red")) {
                loco.stopTrain("RED");
            }

            else if (input.equalsIgnoreCase("danger")) {
                loco.stopTrain("DANGER");
            }
            
            else if (input.equalsIgnoreCase("emergency")) {
            	railway.stopSimulation();
            	loco.stopTrain("EMERGENCY");
            	System.out.println("!!! GLOBAL EMERGENCY SHUTDOWN ACTIVATED!!!");
            }

            else if (input.equalsIgnoreCase("exit")) {
                System.out.println("System Closed.");
                break;
            }

            else {
                System.out.println("Invalid Command : " + input);
            }
        }

        scan.close();
    }
}


package Myrailway;

import uk.ac.leedsbeckett.oop.*;
import javax.swing.JOptionPane; // For Requirement 4: Dialogue boxes
import java.util.ArrayList; // To keep track of locomotives for validation

@SuppressWarnings("serial")
public class MyRailway extends OOPrailwaySim {

    private int rounds = 0;
    private ArrayList<Locomotive> locomotives = new ArrayList<>(); 

    public MyRailway() {
        super();
        setVisible(true);

        System.out.println("===== RAILWAY CONTROL SYSTEM =====");
        System.out.println("Type Commands:");
        System.out.println("--------------------------------");
        System.out.println("about");
        System.out.println("addloco <x> <y>");
        System.out.println("addslowloco <x> <y>");
        System.out.println("attachcarriage <locoId>");
        System.out.println("detachcarriage <locoId>");
        System.out.println("start");
        System.out.println("stop");
        System.out.println("speed <locoId> <speed>");
        System.out.println("crossing <number>");
        System.out.println("reset");
        System.out.println("green / red / danger");
        System.out.println("rounds");
        System.out.println("--------------------------------");
    }

    @Override
    public void processCommand(String input) {
        String[] parts = input.split(" ");
        String command = parts[0].toLowerCase();

        try {
            if (command.equals("about")) {
                about();
            }
            
            else if (command.equals("start")) {
            	
            	startSimulation();
            	displayOutput("Simulation Started");
            }
            
            else if (command.equals("stop")) {
            	
            	stopSimulation();
            	displayOutput("Simulation Stopped");
            }
            
            else if (command.equals("addloco")) {
                if (parts.length == 3) {
                    int x = Integer.parseInt(parts[1]);
                    int y = Integer.parseInt(parts[2]);
                    Locomotive l = new Locomotive(world, x, y);
                    addLocomotive(l);
                    locomotives.add(l); 
                    displayOutput("Locomotive added. ID = " + locomotives.size());
                } else {
                    throw new Exception("Usage: addloco <x> <y>");
                }
            }
            else if (command.equals("addslowloco")) {
                if (parts.length == 3) {
                    int x = Integer.parseInt(parts[1]);
                    int y = Integer.parseInt(parts[2]);
                    NewLoco nl = new NewLoco(this, world, x, y); 
                    addLocomotive(nl);
                    locomotives.add(nl);
                    nl.setLocoId(locomotives.size());
                    displayOutput("Slow Locomotive added. ID = " + locomotives.size());
                } else {
                    throw new Exception("Usage: addslowloco <x> <y>");
                }
            }
            
            else if (command.equals("attachcarriage")) {
                if (parts.length < 2) throw new Exception("Usage: attachcarriage <locoId>");
                int id = Integer.parseInt(parts[1]);
                validateLocoId(id);
                addCarriageToLocomotive(id, new Carriage(world));
                displayOutput("Carriage attached to loco " + id);
            }
            
            else if(input.startsWith("detachcarriage ")) {

                String[] parts1 = input.split(" ");

                if(parts.length == 2) {

                    int locoId = Integer.parseInt(parts[1]);

                    deleteCarriage(locoId);

                    displayOutput("Carriage detached from locomotive : " + locoId);
                }
                else {
                    displayOutput("Usage: detachcarriage <locoId>");
                }
            }
            
         
            
            
            else if (command.equals("speed")) {
                if (parts.length == 3) {
                    int id = Integer.parseInt(parts[1]);
                    int speedVal = Integer.parseInt(parts[2]);
                    validateLocoId(id);
                    locomotives.get(id - 1).setAnimationStepPixels(speedVal);
                    displayOutput("Loco " + id + " speed set to " + speedVal);
                } else {
                    throw new Exception("Usage: speed <locoId> <speed>");
                }
            }
            
            else if (command.equals("crossing")) {
                if (parts.length >= 2) {
                    int cNum = Integer.parseInt(parts[1]);
                    toggleCrossing(cNum);
                    displayOutput("Crossing " + cNum + " toggled.");
                } else {
                    throw new Exception("Usage: crossing <number>");
                }
            }
            
            else if (command.equals("green")) {
                startSimulation(); 
                for (Locomotive l : locomotives) {
                    if (l instanceof NewLoco) {
                        ((NewLoco) l).startTrain(); 
                    }
                displayOutput("GREEN SIGNAL: Simulation Started.");
                }
            }
            
            else if (command.equals("red") || command.equals("danger")) {
                stopSimulation(); 
                for (Locomotive l : locomotives) {
                    if (l instanceof NewLoco) {
                        ((NewLoco) l).stopTrain(command.toUpperCase());
                    }
                }
                displayOutput("STOP SIGNAL: Simulation Halted.");
            }
            
            else if (command.equals("rounds")) {
                displayOutput("Total Rounds Completed: " + rounds);
            }
            
            else if (command.equals("emergency")) {
                stopSimulation(); // Freezes the whole world clock
                for (Locomotive l : locomotives) {
                    if (l instanceof NewLoco) {
                        ((NewLoco) l).stopTrain("EMERGENCY");
                    }
                }
                displayOutput("!!! GLOBAL EMERGENCY SHUTDOWN !!!");
            }
            
            else if (command.equals("reset")) {
                resetSimulation();
                rounds = 0;
                locomotives.clear();
                displayOutput("Simulation Reset.");
            }
            
            else if(input.equalsIgnoreCase("help")) {

                displayOutput(
                    "Commands:\n" +
                    "about\n" +
                    "addloco <x> <y>\n" +
                    "addslowloco <x> <y>\n" +
                    "addcar <locoId>\n" +
                    "detachcarriage <locoId>\n" +
                    "start\n" +
                    "stop\n" +
                    "speed <number>\n" +
                    "cross <number>\n" +
                    "reset\n" +
                    "green\n" +
                    "red\n" +
                    "danger\n" +
                    "rounds"
                );
            }
            
            else {
                displayOutput("Invalid Command: " + input);
            }
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "Error: Please enter valid numbers.");
            
        } catch (Exception e) {
            
            javax.swing.JOptionPane.showMessageDialog(this, e.getMessage());
        }
    }

   
    private void validateLocoId(int id) throws Exception {
        if (id < 1 || id > locomotives.size()) {
            throw new Exception("Invalid ID! There are only " + locomotives.size() + " locomotives.");
        }
    }

    public void incrementRounds() {
        this.rounds++;
        displayOutput("ROUND COMPLETED! Total: " + rounds);
    }

    @Override
    public void about() {
        super.about();
        displayOutput("Railway Simulation System\nAssignment By : Swornalata Chaudhary\nStudent ID : c7682981");
    }
}

package Myrailway;

import uk.ac.leedsbeckett.oop.Locomotive;
import uk.ac.leedsbeckett.oop.GameWorld;

public class NewLoco extends Locomotive {

    public int locoId = -1;

    // Traffic Light States
    private boolean stopped = false;

    // Count completed rounds
    private int rounds = 0;

    // Store previous position
    private String lastPosition = "";
    
    private MyRailway parent;
    

    public NewLoco(MyRailway parent, GameWorld world, int x, int y) {
        super(world, x, y);
        this.parent = parent;
        setAnimationStepPixels(2);
    }

    public void setLocoId(int id) {
        this.locoId = id;
    }
    public int getRounds() {
        return rounds;
    }

    // RED or DANGER = STOP
    public void stopTrain(String reason) {
        stopped = true;
        System.out.println(reason + " SIGNAL -> TRAIN STOPPED");
    }

    // GREEN = MOVE
    public void startTrain() {
        stopped = false;
        System.out.println("GREEN SIGNAL -> TRAIN MOVING");
    }

    @Override
    public String toString() {
    	return "Loco ID: " + locoId + " | Position: " + getCellPosition();
    }
    
 
    public void incrementRounds() {
        this.rounds++;
        System.out.println("ROUND COMPLETED! Total: " + rounds);
    }
    @Override
    public void tick() {

        // Train only moves if not stopped
        if (!stopped) {
            super.tick();
            
            System.out.println(this.toString());
        }
        
        String pos = getCellPosition().toString();
        if (pos.contains("x=7")|| pos.contains("x=8")) {
        	System.out.println("⚠️ TRAIN" + locoId + "APPROACHING CROSSING - CHECK SIGNALS!");
        }
        
        String currentPosition = getCellPosition().toString();

        // Detect one full round
        if (!lastPosition.equals("") &&
            currentPosition.equals("java.awt.Point[x=4,y=3]")) {

            parent.incrementRounds();
            
            System.out.println("ROUND COMPLETED -> " + rounds);
        }

        lastPosition = currentPosition;

        System.out.println(this.toString());
    }
}



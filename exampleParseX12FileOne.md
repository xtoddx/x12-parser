# X12 Parsing with Loop detection.

# Introduction #

Example of parsing a X12 file.

# Details #

There are two steps
  1. Load/Create the X12 configuration object (based on the X12 transaction) to be parsed.
  1. Parse the X12 file.

### Load/Create a configuration object ###

Example of creating an X12 configuration object for a 835 transaction.
```

private static Cf loadCf() {

    Cf cfX12 = new Cf("X12");

    Cf cfISA = cfX12.addChild("ISA", "ISA");
    Cf cfGS = cfISA.addChild("GS", "GS");
    Cf cfST = cfGS.addChild("ST", "ST", "835", 1);

    cfST.addChild("1000A", "N1", "PR", 1);
    cfST.addChild("1000B", "N1", "PE", 1);

    Cf cf2000 = cfST.addChild("2000", "LX");

    Cf cf2100 = cf2000.addChild("2100", "CLP");
    cf2100.addChild("2110", "SVC");

    cfISA.addChild("GE", "GE");
    cfX12.addChild("IEA", "IEA");

    //System.out.println(cfX12);
    return cfX12;
}
```
For more details on creating configuration object check [Creating X12 configuration](https://code.google.com/p/x12-parser/wiki/exampleConfigurationLoading).

### Parse the X12 file ###

```

// The configuration Cf can be loaded using DI framework.
// Check the sample spring application context file provided.

Cf cf835 = loadCf();
Parser parser = new X12Parser(cf835);
X12 x12 = (X12) parser.parse(new File("C:\\test\\835.txt"));


Double totalChargeAmount = 0.0;

// Calculate the total charge amount
List<Loop> loops = x12.findLoop("2100");

for (Loop loop : loops) {
    for (Segment s : loop) {
        if (s.getElement(0).equals("CLP")) {
            totalChargeAmount = totalChargeAmount + Double.parseDouble(s.getElement(3));
        }
    }
}
System.out.println("Total Charged Amount = " + totalChargeAmount.toString());
```


Don't need the loops. Alternate way.
```
Double totalChargeAmount = 0.0;

List<Segment> segments = x12.findSegment("CLP");

for (Segment s : segments) {
    totalChargeAmount = totalChargeAmount + Double.parseDouble(s.getElement(3));
}
System.out.println("Total Charged Amount = " + totalChargeAmount.toString());
```
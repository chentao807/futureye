## Futureye Example 1 ##

### Question ###
<img src='https://lh6.googleusercontent.com/_Cil2MFH7iLM/TN19jeWDEdI/AAAAAAAAABg/WI64bT_jUAY/s800/FutureEyeFirstTest2.png.jpg' />

### Solution ###
Mesh, contour and 3D view:
<img src='https://lh5.googleusercontent.com/_Cil2MFH7iLM/TN19jH3fdUI/AAAAAAAAABc/bjKllifWW0g/s288/FutureEyeFirstTest.png.jpg' /><img src='https://lh3.googleusercontent.com/_Cil2MFH7iLM/TN19j0Dy4pI/AAAAAAAAABk/OTdlyX_Paio/s288/FutureEyeFirstTest3D.png.jpg' />

### Code ###

```
package edu.uta.futureye.tutorial;

import java.util.HashMap;

import edu.uta.futureye.algebra.Solver;
import edu.uta.futureye.algebra.intf.Matrix;
import edu.uta.futureye.algebra.intf.Vector;
import edu.uta.futureye.core.Mesh;
import edu.uta.futureye.core.NodeType;
import edu.uta.futureye.function.basic.FC;
import edu.uta.futureye.function.basic.FX;
import edu.uta.futureye.function.intf.Function;
import edu.uta.futureye.function.operator.FOBasic;
import edu.uta.futureye.io.MeshReader;
import edu.uta.futureye.io.MeshWriter;
import edu.uta.futureye.lib.assembler.AssemblerScalar;
import edu.uta.futureye.lib.element.FELinearTriangle;
import edu.uta.futureye.lib.weakform.WeakFormLaplace2D;
import edu.uta.futureye.util.list.ElementList;

/**
 * Problem:
 *   -\Delta{u} = f
 *   u(x,y)=0, (x,y) \in \partial{\Omega}
 * where
 *   \Omega = [-3,3]*[-3,3]
 *   f = -2*(x^2+y^2)+36
 * Solution:
 *   u = (x^2-9)*(y^2-9)
 * 
 * @author Yueming Liu
 */
public class Laplace {
    public static void main(String[] args) {
        //1.Read in a triangle mesh from an input file with
        //  format ASCII UCD generated by Gridgen
        MeshReader reader = new MeshReader("triangle.grd");
        Mesh mesh = reader.read2DMesh();
        //Compute geometry relationship of nodes and elements
        mesh.computeNodeBelongsToElements();

        //2.Mark border types
        HashMap<NodeType, Function> mapNTF =
                new HashMap<NodeType, Function>();
        mapNTF.put(NodeType.Dirichlet, null);
        mesh.markBorderNode(mapNTF);

        //3.Use element library to assign degrees of
        //  freedom (DOF) to element
        ElementList eList = mesh.getElementList();
        FELinearTriangle feLT = new FELinearTriangle();
        for(int i=1;i<=eList.size();i++)
            feLT.assignTo(eList.at(i));

        //4.Weak form
        WeakFormLaplace2D weakForm = new WeakFormLaplace2D();
        //Right hand side(RHS): f = -2*(x^2+y^2)+36
        Function fx = FX.fx;
        Function fy = FX.fy;
        weakForm.setF(FC.c(-2.0).M( fx.M(fx).A(fy.M(fy)) )
                .A(FC.c(36.0)));

        //5.Assembly process
        AssemblerScalar assembler =
                new AssemblerScalar(mesh, weakForm);
        System.out.println("Begin Assemble...");
        assembler.assemble();
        Matrix stiff = assembler.getStiffnessMatrix();
        Vector load = assembler.getLoadVector();
        //Boundary condition
        assembler.imposeDirichletCondition(FC.c0);
        System.out.println("Assemble done!");

        //6.Solve linear system
        SolverJBLAS solver = new SolverJBLAS();
        Vector u = solver.solveDGESV(stiff, load);
        System.out.println("u=");
        for(int i=1;i<=u.getDim();i++)
            System.out.println(String.format("%.3f", u.get(i)));

        //7.Output results to an Techplot format file
        MeshWriter writer = new MeshWriter(mesh);
        writer.writeTechplot("tuitoral_Laplace.dat", u);
    }
}
```


### Grid File ###
```
"triangle.grd", see Release-1.0
```
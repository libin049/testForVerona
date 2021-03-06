# testForVerona
```C++
// Copyright Microsoft and Project Verona Contributors.
// SPDX-License-Identifier: MIT
class CownTmp {
  tmp: cown[U64Obj];
  
  create(): CownTmp
  {
    var p = new CownTmp;
    p
  }
}

class Fib
{
  /**
   * This example illustrates creating a lot of recursive work by calculating
   * fibonacci numbers top down.
   * 
   * It uses promises to allow results to be reacted to.
   * 
   * The compiler does not have syntax support for promises, and they are just
   * treated as a special type of cown at the moment. This will change but the
   * code illustrates roughly how they would be used.
   * 
   * Cown's only currently wrap object types, not primitives, so the example
   * uses a U64Obj, that boxes a U64.  This is not essential, and will be
   * addressed in the future.
   * 
   * The idealised syntax we are aiming for is given below
   *  fib(x: U64): promise[U64]
      {
        if (x < 2)
        {
          // Implicitely lifting a Value to a promise
          1
        } 
        else
        {
          // When returns a promise for the value it evaluates to.
          // Thus nested when would require an implicit coercion from 
          //     promise[promise[U64]] to promise[U64]
          when () 
          {
            var p1 = Fib.fib(x - 1);
            var p2 = Fib.fib(x - 2);
            when (p1, p2) {
              p1 + p2
            }
          }
        }
      }
   */

  fib(x: U64 & imm): cown[U64Obj] & imm
  {
    var pw = Promise.create();
    var pr = (mut-view pw).wait_handle();

    if (x < 2)
    {
      pw.fulfill(U64Obj.create(1));
    } 
    else
    {
      // `when` with no arguments used to start a new asynchronous task.
      // Better syntax may be needed for this.
      // This is required, so that the whole promise graph isn't constructed
      // sequentially.
      when () 
      {
        var p1 = Fib.fib(x - 1);
        var p2 = Fib.fib(x - 2);
        // When both computations complete, fulfill the result with the addition
        // of the two sub calls.
        when (p1, p2) {
          var r = U64Obj.create(p1.v + p2.v);
          pw.fulfill(r)
        }
      }
    };
    pr
  }

  fib2(x: U64 & imm): cown[U64Obj] & imm
  {
    var pw = Promise.create();
    var pr = (mut-view pw).wait_handle();

    /**if (x < 2)
    {
      pw.fulfill(U64Obj.create(1));
    } 
    else
    {
      var pp1 = when () 
      {
        var p1 = Fib.fib(x - 1);
      };
      var pp2 = when () {
        var p2 = Fib.fib(x - 2);
      };
      when (pp1, pp2) {
        var r = U64Obj.create(pp1.v + pp2.v);
        pw.fulfill(r)
      };
    };*/
    pw.fulfill(U64Obj.create(1));
    when(pr) {
      pr.v = 10;
      var r = U64Obj.create(pr.v);
    };
    pr
  }
 sample_pingpong_pipe() {
      var BUFFER_NUM = 3;
      var x10 = cown.create(U64Obj.create(0));
      var x11 = cown.create(U64Obj.create(0));
      var x12 = cown.create(U64Obj.create(0));
      var x20 = cown.create(U64Obj.create(0));
      var x21 = cown.create(U64Obj.create(0));
      var x22 = cown.create(U64Obj.create(0));
      var x30 = cown.create(U64Obj.create(0));
      var x31 = cown.create(U64Obj.create(0));
      var x32 = cown.create(U64Obj.create(0));
      var i = 0;
      while(i<10) {
        var pingpong = i%BUFFER_NUM;
        if (pingpong == 0) {
          when(x10) {
             x10.v = x10.v+1;
             Builtin.print2("task A{}, x10 = {}\n", i, x10.v);
          };
          when(x10, x20) {
                x20.v = x10.v;
                Builtin.print2("task B{}, x20 = {}\n", i, x20.v);
          };
          when(x20, x30) {
             x30.v = x20.v;
             Builtin.print2("task C{}, x30 = {}\n", i, x30.v);
          };
        };
        if (pingpong == 1) {
          when(x11) {
             x11.v = x11.v+1;
             Builtin.print2("task A{}, x11 = {}\n", i, x11.v);
          };
          when(x11, x21) {
                x21.v = x11.v;
                Builtin.print2("task B{}, x21 = {}\n", i, x21.v);
          };
          when(x21, x31) {
             x31.v = x21.v;
             Builtin.print2("task C{}, x31 = {}\n", i, x31.v);
          };
        };
        if (pingpong == 2) {
          when(x12) {
             x12.v = x12.v+1;
             Builtin.print2("task A{}, x12 = {}\n", i, x12.v);
          };
          when(x12, x22) {
                x22.v = x12.v;
                Builtin.print2("task B{}, x21 = {}\n", i, x22.v);
          };
          when(x22, x32) {
             x32.v = x22.v;
             Builtin.print2("task C{}, x32 = {}\n", i, x32.v);
          };
        };
        i=i+1;
      };
  }
}


class Main
{
  main()
  {
    /**when (var uo = Fib.fib(10))
    {
      // CHECK-L: result=233
      Builtin.print1("result={}\n", uo.v);
    };
    var c1 = cown.create(U64Obj.create(1));
    when(c1) {

    };
    var c2 = cown.create(U64Obj.create(1));
    var a1 = U64Obj.create(1);
    var b1 = a1; //@1
    // a1.v = 10; //error, a1 was previously consumed @1
    when(c2) {// @2
      b1.v = 10;
      Builtin.print1("b1.v={}\n", b1.v);
    };
    // b1.v = 20; //error, b1 was previously consumed @2
    */
    var a2 = 1;
    var c1 = cown.create(U64Obj.create(1));
    when(c1) {
      Builtin.print1("1. a2={}\n", a2);
      a2 = a2+10;
      Builtin.print1("2. a2={}\n", a2);
      c1.v = c1.v+10;
      Builtin.print1("c1.v={}\n", c1.v);
    };
    when(c1) {
      Builtin.print1("3. a2={}\n", a2);
      a2 = a2+20;
      Builtin.print1("4. a2={}\n", a2);
      c1.v = c1.v+20;
      Builtin.print1("c1.v={}\n", c1.v);
    };
    /**var pw = Promise.create();
    var pr = (mut-view pw).wait_handle();
    when () {
      var r = U64Obj.create(1);
      pw.fulfill(r)
    };*/
    /**var a3 = U64Obj.create(1);
    when (a3) {
       
    }*/
    Fib.sample_pingpong_pipe();
  }
}
```
![image](https://user-images.githubusercontent.com/7247717/164442299-b7940fa9-5539-4027-b375-2ee319e2bf96.png)


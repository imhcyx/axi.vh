# axi.vh
Verilog header for easier AXI interface declaration &amp; connection

Include this header file to use AXI interface declaration & connection macros:

```verilog
`include "axi.vh"
```

And make sure axi.vh is in the include directories of your project.

## Declare AXI ports

The following macros are used to declare AXI interfaces in module ports:

```verilog
// AXI4-Lite ports
`AXI4LITE_SLAVE_IF  (prefix, addr_width, data_width)
`AXI4LITE_MASTER_IF (prefix, addr_width, data_width)

// AXI4 ports
`AXI4_SLAVE_IF  (prefix, addr_width, data_width, id_width)
`AXI4_MASTER_IF (prefix, addr_width, data_width, id_width)

// AXI4 ports without ID signals
`AXI4_SLAVE_IF_NO_ID    (prefix, addr_width, data_width)
`AXI4_MASTER_IF_NO_ID   (prefix, addr_width, data_width)
```

Example:

```verilog
module example (
    input  wire         clk,
    input  wire         rst,
    `AXI4_MASTER_IF     (m_axi, 32, 64, 4), // be careful with the comma in the end
    /* The macro above expands to:
    output wire         m_axi_awvalid,
    input  wire         m_axi_awready,
    output wire [31:0]  m_axi_awaddr,
    output wire [2:0]   m_axi_awprot,
    output wire [3:0]   m_axi_awid,
    output wire [7:0]   m_axi_awlen,
    output wire [2:0]   m_axi_awsize,
    output wire [1:0]   m_axi_awburst,
    output wire         m_axi_awlock,
    output wire [3:0]   m_axi_awcache,
    output wire [3:0]   m_axi_awqos,
    output wire [3:0]   m_axi_awregion,
    ... and AR, W, R, B channel signals
    */
    `AXI4LITE_SLAVE_IF  (s_axilite, 12, 32) // no comma for the last declaration
    // Similar expansion results ...
);

    // ...

endmodule
```

## Define AXI interface wires

If you need to declare wires in module context for AXI connection, use the following macros:

```verilog
// AXI4-Lite wires
`AXI4LITE_WIRE(prefix, addr_width, data_width)

// AXI4 wires
`AXI4_WIRE(prefix, addr_width, data_width, id_width)

// AXI4 wires without ID signals
`AXI4_WIRE_NO_ID(prefix, addr_width, data_width)
```

Example:

```verilog
module example;

    `AXI4_WIRE(axi_conn, 32, 64, 4); // semicolon is required
    /* The macro above expands to:
    wire            axi_conn_awvalid;
    wire            axi_conn_awready;
    wire [31:0]     axi_conn_awaddr;
    wire [2:0]      axi_conn_awprot;
    wire [3:0]      axi_conn_awid;
    wire [7:0]      axi_conn_awlen;
    wire [2:0]      axi_conn_awsize;
    wire [1:0]      axi_conn_awburst;
    wire            axi_conn_awlock;
    wire [3:0]      axi_conn_awcache;
    wire [3:0]      axi_conn_awqos;
    wire [3:0]      axi_conn_awregion;
    ...
    */

endmodule
```

## Connect AXI interfaces

The following macros are used to connect AXI ports in the port list of a module instantiation:

```verilog
// AXI4-Lite connection
`AXI4LITE_CONNECT(port_prefix, outer_prefix)

// AXI4 connection
`AXI4_CONNECT(port_prefix, outer_prefix)

// AXI4 connection without ID signals
`AXI4_CONNECT_NO_ID(port_prefix, outer_prefix)
```

These can be used to connect inner AXI interfaces to outer:

```verilog
module submodule (
    // ...
    `AXI4_MASTER_IF(m_axi, 32, 32, 1)
);
    // ...
endmodule

module example (
    // ...
    `AXI4_MASTER_IF(m_axi_outer, 32, 32, 1)
);

    submodule u_sub (
        // ...
        `AXI4_CONNECT(m_axi, m_axi_outer)
        /* The macro above expands to:
        .m_axi_awvalid  (m_axi_outer_awvalid),
        .m_axi_awready  (m_axi_outer_awready),
        ...
        */
    );

endmodule
```

Or connect two submodules:

```verilog
module submodule1 (
    // ...
    `AXI4_SLAVE_IF(s_axi, 32, 32, 1)
);
    // ...
endmodule

module submodule2 (
    // ...
    `AXI4_MASTER_IF(m_axi, 32, 32, 1)
);
    // ...
endmodule

module example;

    `AXI4_WIRE(axi_conn, 32, 32, 1);

    submodule1 u_sub1 (
        // ...
        `AXI4_CONNECT(s_axi, axi_conn)
    );

    submodule2 u_sub2 (
        // ...
        `AXI4_CONNECT(m_axi, axi_conn)
    );

endmodule
```

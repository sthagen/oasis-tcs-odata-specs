# ##sec Introduction

This specification adds aggregation functionality to the Open Data Protocol (OData) without changing any of the base principles of OData. It defines semantics and a representation for aggregation of data, especially:
- Semantics and operations for querying aggregated data,
- Results format for queries containing aggregated data,
- Vocabulary terms to annotate what can be aggregated, and how.

## ##subsec Glossary

### ##subsubsec Definitions of Terms

This specification defines the following terms:
- <a name="AggregatableExpression">_Aggregatable Expression_</a> – an [expression](#Expression) resulting in a value of an [aggregatable primitive type](#AggregatablePrimitiveType)
- <a name="AggregatablePrimtiveType">_Aggregatable Primitive Type_</a> – a primitive type other than `Edm.Stream` or subtypes of `Edm.Geography` or `Edm.Geometry`
- <a name="DataAggregationPath">_Data Aggregation Path_</a> – a path that consists of one or more segments separated by a forward slash. Segments are names of declared or dynamic structural or navigation properties, or type-cast segments consisting of the (optionally qualified) name of a structured type that is derived from the type identified by the preceding path segment to reach properties declared by the derived type.
- <a name="Expression">_Expression_</a> – derived from the `commonExpr` rule (see [OData-ABNF](#ODataABNF)). Certain expressions are evaluated relative to a "current instance", while other expressions introduced in this document are evaluated to a "current collection", see [example ##collexpr].
- <a name="SingleValuedPropertyPath">_Single-Valued Property Path_</a> – property path ending in a single-valued primitive, complex, or navigation property

### ##subsubsec Acronyms and Abbreviations

- $A,I,U$ – collections of instances
- $H$ – hierarchical collection
- $u,w$ – instances in a collection
- $x,y$ – instances in a hierarchical collection, called nodes
- $p,q,v$ – paths
- $S,T$ – transformation sequences
- $\Gamma(A,v)$ – the collection that results from evaluating a [data aggregation path](#DataAggregationPath) $v$ relative to a collection $A$, defined [below](#EvaluationofDataAggregationPaths)
- $γ(u,v)$ – the collection that results from evaluating a [data aggregation path](#DataAggregationPath) $v$ relative to an instance $u$, defined [below](#EvaluationofDataAggregationPaths)
- $\Pi_G(s)$ – a transformation of a collection that injects grouping properties into every instance of the collection, defined [below](#SimpleGrouping)

### ##subsubsec Document Conventions

Keywords defined by this specification use `this monospaced font`.

$$\hbox{\tt Normative source code uses this paragraph style.}$$

Some sections of this specification are illustrated with non-normative examples.

::: example
Example ##ex: text describing an example uses this paragraph style
```
Non-normative examples use this paragraph style.
```
:::

All examples in this document are non-normative and informative only. Examples labeled with ⚠ contain advanced concepts or make use of keywords that are defined only later in the text, they can be skipped at first reading.

All other text is normative unless otherwise labeled.

# ##sec Overview

Open Data (OData) services expose a data model that describes the schema of the service in terms of the Entity Data Model (EDM, see [OData-CSDL](#ODataCDSL)) and then allows for querying data in terms of this model. The responses returned by an OData service are based on that data model and retain the relationships between the entities in the model.

Extending the OData query features with simple aggregation capabilities avoids cluttering OData services with an exponential number of explicitly modeled "aggregation level entities" or else restricting the consumer to a small subset of predefined aggregations.

Adding the notion of aggregation to OData without changing any of the base principles in OData has two aspects:
1. Means for the consumer to query aggregated data on top of any given data model (for sufficiently capable data providers)
2. Means for the provider to annotate what data can be aggregated, and in which way, allowing consumers to avoid asking questions that the provider cannot answer.

Implementing any of these two aspects is valuable in itself independent of the other, and implementing both provides additional value for consumers. The provided aggregation annotations help a consumer understand more of the data structure looking at the service's exposed data model. The query extensions allow the consumers to explicitly express the desired aggregation behavior for a particular query. They also allow consumers to formulate queries that utilize the aggregation annotations.

## ##subsec Example Data Model

::: example
Example ##ex: The following diagram depicts a simple model that is used throughout this document.

<svg class="st20" viewBox="0 200 600 350" style="width:100%">
  <style type="text/css">
  <![CDATA[
    .st1 {fill:#f2f2f2;stroke:none;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .st2 {fill:#ffffff;stroke:none;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.7}
    .st3 {fill:#000000;font-family:Arial;font-size:0.666664em}
    .st4 {font-size:1em}
    .st5 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .st6 {fill:none;visibility:hidden}
    .st7 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:1.4}
    .st8 {fill:#000000;font-family:Arial;font-size:0.833336em}
    .st9 {fill:#000000;font-family:Arial;font-size:0.833336em;font-style:italic}
    .st10 {fill:none}
    .st11 {marker-start:url(#mrkr1-125);stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75;visibility:hidden}
    .st12 {fill:#000000;fill-opacity:1;stroke:#000000;stroke-opacity:1;stroke-width:0.22935779816514}
    .st13 {fill:#000000;font-family:Arial;font-size:0.75em}
    .st14 {fill:#000000;fill-opacity:1;stroke:#000000;stroke-opacity:1;stroke-width:0.22222222222222}
    .st15 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.72}
    .st16 {marker-end:url(#mrkr1-200);stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.72}
    .st17 {fill:none;stroke:none;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .st18 {fill:#000000;font-family:Calibri;font-size:0.833336em}
    .st19 {marker-end:url(#mrkr14-311);stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .st20 {fill:none;fill-rule:evenodd;font-size:12px;overflow:visible;stroke-linecap:square;stroke-miterlimit:3}
  ]]>
  </style>
  <defs id="Markers">
    <g id="lend1">
      <path d="M 1 -1 L 0 0 L 1 1 "
        style="stroke-linecap:round;stroke-linejoin:round;fill:none" />
    </g>
    <marker id="mrkr1-125" class="st12" orient="auto"
      markerUnits="strokeWidth" overflow="visible">
      <use xlink:href="#lend1" transform="scale(4.36) " />
    </marker>
    <g id="lend2">
      <path d="M 1 1 L 0 0 L 1 -1 L 1 1 " style="stroke:none" />
    </g>
    <marker id="mrkr2-137" class="st14" refX="-4.5" orient="auto"
      markerUnits="strokeWidth" overflow="visible">
      <use xlink:href="#lend2" transform="scale(-4.5,-4.5) " />
    </marker>
    <marker id="mrkr1-200" class="st14" orient="auto"
      markerUnits="strokeWidth" overflow="visible">
      <use xlink:href="#lend1" transform="scale(-4.5,-4.5) " />
    </marker>
    <g id="lend14">
      <path d="M 3 -1 L 0 0 L 3 1 L 3 -1 "
        style="stroke-linecap:round;stroke-linejoin:round;fill:none" />
    </g>
    <marker id="mrkr14-311" class="st12" refX="-13.08"
      orient="auto" markerUnits="strokeWidth" overflow="visible">
      <use xlink:href="#lend14" transform="scale(-4.36,-4.36) " />
    </marker>
  </defs>
  <g id="group1000-1" transform="translate(208.346,-446.457)">
    <g id="shape1001-2" transform="translate(0,-28.3465)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1002-4">
      <rect x="0" y="813.543" width="110.551" height="28.3465"
        class="st2" />
      <text x="4" y="825.32" class="st3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="st4">Amount: Edm.Decimal</tspan>
      </text>
    </g>
    <g id="shape1003-8">
    </g>
    <g id="shape1004-10" transform="translate(-4.79616E-14,-28.3465)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1005-13">
    </g>
    <g id="shape1006-15">
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st6" />
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st7" />
    </g>
    <g id="shape1000-18">
      <text x="45.27" y="805.2" class="st8">Sale</text>
    </g>
  </g>
  <g id="group1007-20" transform="translate(208.346,-566.929)">
    <g id="shape1008-21" transform="translate(0,-48.7559)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1009-23">
      <desc>Date: Edm.Date {id} Month: Edm.String Quarter: Edm.String
        Yea...</desc>
      <rect x="0" y="793.134" width="110.551" height="48.7559"
        class="st2" />
      <text x="4" y="805.51" class="st3">
        Date: Edm.Date {id}
        <tspan x="4" dy="1.2em" class="st4">Month: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="st4">Quarter: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="st4">Year: Edm.Int16</tspan>
      </text>
    </g>
    <g id="shape1010-29">
    </g>
    <g id="shape1011-31" transform="translate(-4.79616E-14,-48.7559)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1012-34">
    </g>
    <g id="shape1013-36">
      <rect x="0" y="770.457" width="110.551" height="71.4331"
        class="st6" />
      <rect x="0" y="770.457" width="110.551" height="71.4331"
        class="st7" />
    </g>
    <g id="shape1007-39">
      <text x="44.16" y="784.8" class="st8">Time</text>
    </g>
  </g>
  <g id="group1014-41" transform="translate(26.7784,-442.205)">
    <g id="shape1015-42" transform="translate(0,-36.8504)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1016-44">
      <rect x="0" y="805.039" width="110.551" height="36.8504"
        class="st2" />
      <text x="4" y="816.26" class="st3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="st4">Name: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="st4">Country: Edm.String</tspan>
      </text>
    </g>
    <g id="shape1017-49">
    </g>
    <g id="shape1018-51" transform="translate(-4.79616E-14,-36.8504)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1019-54">
    </g>
    <g id="shape1020-56">
      <rect x="0" y="782.362" width="110.551" height="59.5276"
        class="st6" />
      <rect x="0" y="782.362" width="110.551" height="59.5276"
        class="st7" />
    </g>
    <g id="shape1014-59">
      <text x="33.6" y="796.7" class="st8">Customer</text>
    </g>
  </g>
  <g id="group1028-61" transform="translate(384.094,-577.134)">
    <g id="shape1029-62" transform="translate(0,-28.3465)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1030-64">
      <rect x="0" y="813.543" width="110.551" height="28.3465"
        class="st2" />
      <text x="4" y="825.32" class="st3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="st4">Name: Edm.String</tspan>
      </text>
    </g>
    <g id="shape1031-68">
    </g>
    <g id="shape1032-70" transform="translate(-4.79616E-14,-28.3465)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1033-73">
    </g>
    <g id="shape1034-75">
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st6" />
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st7" />
    </g>
    <g id="shape1028-78">
      <text x="34.99" y="805.2" class="st8">Category</text>
    </g>
  </g>
  <g id="group1035-80" transform="translate(384.094,-436.252)">
    <g id="shape1036-81" transform="translate(0,-48.7559)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1037-83">
      <desc>ID: Edm.String {id} Name: Edm.String Color: Edm.String
        TaxRat...</desc>
      <rect x="0" y="793.134" width="110.551" height="48.7559"
        class="st2" />
      <text x="4" y="805.51" class="st3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="st4">Name: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="st4">Color: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="st4">TaxRate: Edm.Decimal</tspan>
      </text>
    </g>
    <g id="shape1038-89">
    </g>
    <g id="shape1039-91" transform="translate(-4.79616E-14,-48.7559)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1040-94">
    </g>
    <g id="shape1041-96">
      <rect x="0" y="770.457" width="110.551" height="71.4331"
        class="st6" />
      <rect x="0" y="770.457" width="110.551" height="71.4331"
        class="st7" />
    </g>
    <g id="shape1035-99">
      <text x="38.04" y="784.8" class="st9">Product</text>
    </g>
  </g>
  <g id="group1042-101" transform="translate(208.346,-324)">
    <g id="shape1043-102" transform="translate(0,-28.3465)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1044-104">
      <rect x="0" y="813.543" width="110.551" height="28.3465"
        class="st2" />
      <text x="4" y="825.32" class="st3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="st4">Name: Edm.String</tspan>
      </text>
    </g>
    <g id="shape1045-108">
    </g>
    <g id="shape1046-110" transform="translate(-4.79616E-14,-28.3465)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1047-113">
    </g>
    <g id="shape1048-115">
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st6" />
      <rect x="0" y="790.866" width="110.551" height="51.0236"
        class="st7" />
    </g>
    <g id="shape1042-118">
      <text x="14.42" y="805.2" class="st8">SalesOrganization</text>
    </g>
  </g>
  <g id="group1054-120" transform="translate(208.346,-464.882)">
    <g id="shape1055-121" transform="translate(-70.1169,-22.1102)">
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">1</text>
    </g>
    <g id="shape1056-127" transform="translate(-15.5072,7.65354)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1057-132" transform="translate(0,1673.86) rotate(180)">
    </g>
    <g id="shape1058-134" transform="translate(-47.847,-1.27717)">
    </g>
    <g id="shape1054-138">
      <path d="M0 834.8 L-71.02 834.8" class="st15" />
    </g>
  </g>
  <g id="group1064-141" transform="translate(318.898,-464.882)">
    <g id="shape1065-142" transform="translate(48.9382,-22.1102)">
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">1</text>
    </g>
    <g id="shape1066-147" transform="translate(0.0833553,8.50393)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1067-152" transform="translate(0,-4.25197)">
    </g>
    <g id="shape1068-154" transform="translate(20.2598,-1.27717)">
    </g>
    <g id="shape1064-157">
      <path d="M0 834.8 L65.2 834.8" class="st15" />
    </g>
  </g>
  <g id="group1069-160" transform="translate(432.283,-577.134)">
    <g id="shape1070-161" transform="translate(5.75265,55.559)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1071-166" transform="translate(-10.5894,14.1732)">
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">1</text>
    </g>
    <g id="shape1072-171"
      transform="translate(846.142,841.89) rotate(90)">
    </g>
    <g id="shape1073-173" transform="translate(-5.25197,44.0181)">
    </g>
    <g id="shape1069-176">
      <path d="M7.09 841.89 L7.09 911.34" class="st15" />
    </g>
  </g>
  <g id="group1079-179" transform="translate(263.622,-324)">
    <g id="shape1080-180" transform="translate(67.1811,-12.7559)">
      <rect x="0" y="830.551" width="22.6772" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="22.6772" height="11.3386"
        class="st11" />
      <text x="3.83" y="838.92" class="st13">0..1</text>
    </g>
    <g id="shape1081-185" transform="translate(-14.0899,14.1732)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1082-190"
      transform="translate(839.055,841.89) rotate(90)">
    </g>
    <g id="shape1083-192" transform="translate(15.2992,-7.4189)">
    </g>
    <g id="shape1079-195">
      <path
        d="M0 841.89 L-0 852.52 A8.50394 8.50394 -180 0 0 8.5 861.02 L57.4 861.02 A8.50394 8.50394 -180 0 0 65.91 852.52
               L65.91 821.69 A5.31496 5.31496 -180 0 0 60.59 816.38 L55.28 816.38"
        class="st16" />
    </g>
  </g>
  <g id="group1084-201" transform="translate(256.535,-446.457)">
    <g id="shape1085-202" transform="translate(-6.90433,69.4488)">
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">1</text>
    </g>
    <g id="shape1086-207" transform="translate(5.75264,14.4567)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1087-212"
      transform="translate(846.142,841.89) rotate(90)">
    </g>
    <g id="shape1088-214" transform="translate(-5.25197,45.0929)">
    </g>
    <g id="shape1084-217">
      <path d="M7.09 841.89 L7.09 913.32" class="st16" />
    </g>
  </g>
  <g id="group1094-222" transform="translate(256.535,-497.48)">
    <g id="shape1095-223" transform="translate(-9.17205,-52.441)">
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="15.5095" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">1</text>
    </g>
    <g id="shape1096-228" transform="translate(7.16997,0.708661)">
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st10" />
      <rect x="0" y="830.551" width="14.0065" height="11.3386"
        class="st11" />
      <text x="5.25" y="838.92" class="st13">*</text>
    </g>
    <g id="shape1097-233"
      transform="translate(-831.969,841.89) rotate(-90)">
    </g>
    <g id="shape1098-235" transform="translate(-5.25197,-31.2181)">
    </g>
    <g id="shape1094-238">
      <path d="M7.09 841.89 L7.09 772.44" class="st16" />
    </g>
  </g>
  <g id="shape1099-243" transform="translate(173.622,-446.457)">
    <rect x="0" y="827.717" width="35.4331" height="14.1732"
      class="st17" />
    <text x="7.43" y="837.8" class="st18">Sales</text>
  </g>
  <g id="shape1100-246" transform="translate(317.126,-446.457)">
    <rect x="0" y="827.717" width="35.4331" height="14.1732"
      class="st17" />
    <text x="7.43" y="837.8" class="st18">Sales</text>
  </g>
  <g id="shape1101-249" transform="translate(138.114,-474.803)">
    <rect x="0" y="827.717" width="49.7571" height="14.1732"
      class="st17" />
    <text x="5.09" y="837.8" class="st18">Customer</text>
  </g>
  <g id="shape1102-252" transform="translate(336.539,-474.803)">
    <rect x="0" y="827.717" width="49.7571" height="14.1732"
      class="st17" />
    <text x="8.87" y="837.8" class="st18">Product</text>
  </g>
  <g id="shape1103-255" transform="translate(436.385,-510.236)">
    <rect x="0" y="827.717" width="49.7571" height="14.1732"
      class="st17" />
    <text x="6.92" y="837.8" class="st18">Products</text>
  </g>
  <g id="shape1104-258" transform="translate(389.613,-552.756)">
    <rect x="0" y="827.717" width="49.7571" height="14.1732"
      class="st17" />
    <text x="6.66" y="837.8" class="st18">Category</text>
  </g>
  <g id="shape1105-261" transform="translate(233.707,-538.583)">
    <rect x="0" y="827.717" width="49.7571" height="14.1732"
      class="st17" />
    <text x="4" y="837.8" class="st18">Time</text>
  </g>
  <g id="shape1106-264" transform="translate(183.366,-388.878)">
    <rect x="0" y="827.717" width="92.126" height="14.1732"
      class="st17" />
    <text x="4" y="837.8" class="st18">SalesOrganization</text>
  </g>
  <g id="shape1108-267" transform="translate(330.236,-324)">
    <rect x="0" y="827.717" width="69.7039" height="14.1732"
      class="st17" />
    <text x="4" y="837.8" class="st18">Superordinate</text>
  </g>
  <g id="group1110-270" transform="translate(432.283,-368.504)">
    <g id="shape1111-271" transform="translate(0,-20.4094)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1112-273">
      <rect x="0" y="821.48" width="110.551" height="20.4094"
        class="st2" />
      <text x="4" y="834.09" class="st3">Rating: Edm.Byte</text>
    </g>
    <g id="shape1113-276">
    </g>
    <g id="shape1114-278" transform="translate(-4.79616E-14,-20.4094)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1115-281">
    </g>
    <g id="shape1116-283">
      <rect x="0" y="798.803" width="110.551" height="43.0866"
        class="st6" />
      <rect x="0" y="798.803" width="110.551" height="43.0866"
        class="st7" />
    </g>
    <g id="shape1110-286">
      <text x="26.65" y="813.14" class="st8">FoodProduct</text>
    </g>
  </g>
  <g id="group1117-288" transform="translate(432.283,-308.153)">
    <g id="shape1118-289" transform="translate(0,-20.4094)">
      <rect x="0" y="819.213" width="110.551" height="22.6772"
        class="st1" />
    </g>
    <g id="shape1119-291">
      <rect x="0" y="821.48" width="110.551" height="20.4094"
        class="st2" />
      <text x="4" y="834.09" class="st3">RatingClass: Edm.String</text>
    </g>
    <g id="shape1120-294">
    </g>
    <g id="shape1121-296" transform="translate(-4.79616E-14,-20.4094)">
      <path d="M0 841.89 L110.55 841.89" class="st5" />
    </g>
    <g id="shape1122-299">
    </g>
    <g id="shape1123-301">
      <rect x="0" y="798.803" width="110.551" height="43.0866"
        class="st6" />
      <rect x="0" y="798.803" width="110.551" height="43.0866"
        class="st7" />
    </g>
    <g id="shape1117-304">
      <text x="17.48" y="813.14" class="st8">NonFoodProduct</text>
    </g>
  </g>
  <g id="shape1124-306" transform="translate(432.283,-390.047)">
    <path d="M0 841.89 L-20.55 841.89 L-20.55 805.5" class="st19" />
  </g>
  <g id="shape1125-312" transform="translate(432.283,-329.697)">
    <path d="M0 841.89 L-20.55 841.89 L-20.55 745.14" class="st19" />
  </g>
</svg>

The Amount property in the Sale entity type is an [aggregatable property](#AggregationCapabilities), and the properties of the related entity types are groupable. These can be arranged in four hierarchies:
- Product hierarchy based on [groupable](#AggregationCapabilities) properties of the Category and Product entity types
- Customer [hierarchy](#LeveledHierarchy) based on Country and Customer
- Time [hierarchy](#LeveledHierarchy) based on Year, Month and Date
- SalesOrganization [hierarchy](#RecursiveHierarchy) based on the recursive association to itself

In the context of Online Analytical Processing (OLAP), this model might be described in terms of a Sales "cube" with an Amount "measure" and three "dimensions". This document will avoid such terms, as they are heavily overloaded.
:::

Query extensions and descriptive annotations can both be applied to normalized as well as partly or fully denormalized schemas.

::: example
Example ##ex: The following diagram depicts a denormalized schema for the simple model.

<svg viewBox="100 150 600 250" class="dst11" style="width:100%">
  <style type="text/css">
  <![CDATA[
    .dst1 {fill:#f2f2f2;stroke:none;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .dst2 {fill:#ffffff;stroke:none;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.7}
    .dst3 {fill:#000000;font-family:Arial;font-size:0.666664em}
    .dst4 {font-size:1em}
    .dst5 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.75}
    .dst6 {fill:none;visibility:hidden}
    .dst7 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:1.4}
    .dst8 {fill:#000000;font-family:Arial;font-size:0.833336em}
    .dst9 {stroke:#000000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.24}
    .dst10 {fill:#000000;font-family:Calibri;font-size:0.666664em}
    .dst11 {fill:none;fill-rule:evenodd;font-size:12px;overflow:visible;stroke-linecap:square;stroke-miterlimit:3}
  ]]>
  </style>
  <g id="group1000-1" transform="translate(164.409,-456.378)">
    <g id="shape1001-2" transform="translate(0,-198.425)">
      <rect x="0" y="819.213" width="178.583" height="22.6772"
        class="dst1" />
    </g>
    <g id="shape1002-4">
      <rect x="0" y="643.465" width="178.583" height="198.425"
        class="dst2" />
      <text x="4" y="653.88" class="dst3">
        ID: Edm.String {id}
        <tspan x="4" dy="1.2em" class="dst4">Amount: Edm.Decimal </tspan>
        <tspan x="4" dy="1.2em" class="dst4">CategoryID: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">CategoryName: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">ProductID: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">ProductName: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">ProductColor: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">ProductTaxRate: Edm.Decimal </tspan>
        <tspan x="4" dy="1.2em" class="dst4">FoodProductRating: Edm.Byte </tspan>
        <tspan x="4" dy="1.2em" class="dst4">NonFoodProductRatingClass:
          Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">SalesOrganizationID: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">SalesOrganizationName:
          Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">SalesOrganizationSuperordinateID:
          Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">TimeDate: Edm.Date </tspan>
        <tspan x="4" dy="1.2em" class="dst4">TimeMonth: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">TimeQuarter: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">TimeYear: Edm.Int16 </tspan>
        <tspan x="4" dy="1.2em" class="dst4">CustomerID: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">CustomerName: Edm.String </tspan>
        <tspan x="4" dy="1.2em" class="dst4">CustomerCountry: Edm.String</tspan>
      </text>
    </g>
    <g id="shape1003-26">
    </g>
    <g id="shape1004-28" transform="translate(-1.43885E-13,-198.425)">
      <path d="M0 841.89 L178.58 841.89" class="dst5" />
    </g>
    <g id="shape1005-31">
    </g>
    <g id="shape1006-33">
      <rect x="0" y="620.787" width="178.583" height="221.102"
        class="dst6" />
      <rect x="0" y="620.787" width="178.583" height="221.102"
        class="dst7" />
    </g>
    <g id="shape1000-36">
      <text x="79.28" y="635.13" class="dst8">Sale</text>
    </g>
  </g>
  <g id="shape1011-38" transform="translate(987.52,192.756) rotate(90)">
    <path
      d="M0 827.01 L0 838.35 A3.54325 3.54325 -180 0 0 3.54 841.89 A3.54325 3.54325 0 0 1 7.09 845.43 A3.54325 3.54325
             0 0 1 10.63 841.89 A3.54325 3.54325 -180 0 0 14.17 838.35 L14.17 827.01"
      class="dst9" />
    <text x="-865.89" y="9.49" transform="rotate(-90)" class="dst10">Sales</text>
  </g>
  <g id="shape1012-42"
    transform="translate(994.961,211.181) rotate(90)">
    <path
      d="M0 834.45 L0 838.35 A3.54325 3.54325 -180 0 0 3.54 841.89 A3.54325 3.54325 0 0 1 7.09 845.43 A3.54325 3.54325
             0 0 1 10.63 841.89 A3.54325 3.54325 -180 0 0 14.17 838.35 L14.17 834.45"
      class="dst9" />
    <text x="-878.59" y="9.49" transform="rotate(-90)" class="dst10">Category</text>
  </g>
  <g id="shape1013-46"
    transform="translate(994.961,229.606) rotate(90)">
    <path
      d="M0 834.45 L0 837.99 A3.89757 3.89757 -180 0 0 3.9 841.89 L13.11 841.89 A3.89757 3.89757 0 0 1 17.01 845.79 A3.89757
             3.89757 0 0 1 20.91 841.89 L30.12 841.89 A3.89757 3.89757 -180 0 0 34.02 837.99 L34.02 834.45"
      class="dst9" />
    <text x="-875.4" y="19.41" transform="rotate(-90)" class="dst10">Product</text>
  </g>
  <g id="shape1014-50"
    transform="translate(994.961,267.874) rotate(90)">
    <path
      d="M0 834.45 L0 840.12 A1.77162 1.77162 -180 0 0 1.77 841.89 A1.77162 1.77162 0 0 1 3.54 843.66 A1.77162 1.77162
             0 0 1 5.31 841.89 A1.77162 1.77162 -180 0 0 7.09 840.12 L7.09 834.45"
      class="dst9" />
    <text x="-863.98" y="5.94" transform="rotate(-90)" class="dst10">Food</text>
  </g>
  <g id="shape1015-54"
    transform="translate(994.961,276.378) rotate(90)">
    <path
      d="M0 834.45 L0 840.12 A1.77162 1.77162 -180 0 0 1.77 841.89 A1.77162 1.77162 0 0 1 3.54 843.66 A1.77162 1.77162
             0 0 1 5.31 841.89 A1.77162 1.77162 -180 0 0 7.09 840.12 L7.09 834.45"
      class="dst9" />
    <text x="-879.37" y="5.94" transform="rotate(-90)" class="dst10">Non Food</text>
  </g>
  <g id="shape1016-58"
    transform="translate(994.961,289.134) rotate(90)">
    <path
      d="M0 834.45 L0 837.99 A3.89757 3.89757 -180 0 0 3.9 841.89 L7.44 841.89 A3.89757 3.89757 0 0 1 11.34 845.79 A3.89757
             3.89757 0 0 1 15.24 841.89 L18.78 841.89 A3.89757 3.89757 -180 0 0 22.68 837.99 L22.68 834.45"
      class="dst9" />
    <text x="-879.9" y="13.74" transform="rotate(-90)" class="dst10">Sales
      Org</text>
  </g>
  <g id="shape1018-62" transform="translate(994.961,317.48) rotate(90)">
    <path
      d="M0 834.45 L0 837.99 A3.89757 3.89757 -180 0 0 3.9 841.89 L13.11 841.89 A3.89757 3.89757 0 0 1 17.01 845.79 A3.89757
             3.89757 0 0 1 20.91 841.89 L30.12 841.89 A3.89757 3.89757 -180 0 0 34.02 837.99 L34.02 834.45"
      class="dst9" />
    <text x="-865.89" y="19.41" transform="rotate(-90)" class="dst10">Time</text>
  </g>
  <g id="shape1019-66"
    transform="translate(994.961,357.165) rotate(90)">
    <path
      d="M0 834.45 L0 839.06 A2.8346 2.8346 -180 0 0 2.83 841.89 L8.5 841.89 A2.8346 2.8346 0 0 1 11.34 844.72 A2.8346
             2.8346 0 0 1 14.17 841.89 L19.84 841.89 A2.8346 2.8346 -180 0 0 22.68 839.06 L22.68 834.45"
      class="dst9" />
    <text x="-880.38" y="13.74" transform="rotate(-90)" class="dst10">Customer</text>
  </g>
</svg>
:::

## ##subsec Example Data

::: example
Example ##ex: The following entity sets and sample data will be used to further illustrate the capabilities introduced by this extension.

:::: {.example-data style=width:600px;height:700px}
<svg viewBox="0 0 600 700">
  <defs>
    <marker id="begin" viewBox="0 0 10 10" refX="0" refY="5" orient="auto" markerWidth="5" markerHeight="5">
      <path d="M10,0 L0,5 L10,10 z" />
    </marker>
    <marker id="end" viewBox="0 0 10 10" refX="10" refY="5" orient="auto" markerWidth="5" markerHeight="5">
      <path d="M0,0 L10,5 L0,10 z" />
    </marker>
  </defs>
  <path d="M310,150 l45,50" marker-start="url(#begin)" marker-end="url(#end)" />
  <path d="M50,485 l0,-35" marker-start="url(#begin)" marker-end="url(#end)" />
  <path d="M160,485 l30,-185" marker-end="url(#end)" />
  <path d="M215,485 l45,-335" marker-start="url(#begin)" marker-end="url(#end)" />
  <path d="M300,485 l55,-165" marker-start="url(#begin)" marker-end="url(#end)" />
  <path d="M525,300 l-40,-20" marker-end="url(#end)" />
</svg>

::::: {.nav style=left:250px}
Products

ID|Category|Name|Color|TaxRate
--|--------|----|-----|------:
P1|PG1|Sugar|White|0.06
P2|PG1|Coffee|Brown|0.06
P3|PG2|Paper|White|0.14
P4|PG2|Pencil|Black|0.14
:::::

::::: {style=left:510px}
Food

|Rating|
|------|
|5|
|&nbsp;|
|n/a|
|n/a|
:::::

::::: {style=left:570px}
Non-Food

|RatingClass|
|------|
|n/a|
|n/a|
|average|
|&nbsp;|
:::::

::::: {style=top:150px}
Time

Date|Month|Quarter|Year
----|-----|-------|----
2022-01-01|2022-01|2022-1|2022
2022-04-01|2022-04|2022-2|2022
2022-04-10|2022-04|2022-2|2022
...|||
:::::

::::: {style=top:150px;left:360px}
Categories

ID|Name
--|----
PG1|Food
PG2|Non-Food
:::::

::::: {.nav style=top:260px;left:360px}
Sales Organizations

ID|Superordinate|Name
--|-------------|----
Sales||Corporate Sales
US|Sales|US
US West|US|US West
US East|US|US East
EMEA|Sales|EMEA
EMEA Central|EMEA|EMEA Central
:::::

::::: {style=top:300px}
Customers

ID|Name|Country
--|----|-------
C1|Joe|USA
C2|Sue|USA
C3|Sue|Netherlands
C4|Luc|France
:::::

::::: {.nav .nav-2 style=top:450px}
Sales

ID|Customer|Time|Product|Sales Organization|Amount
--|--------|----|-------|------------------|-----:
1|C1|2022-01-03|P3|US West|1
2|C1|2022-04-10|P1|US West|2
3|C1|2022-08-07|P2|US West|4
4|C2|2022-01-03|P2|US East|8
5|C2|2022-11-09|P3|US East|4
6|C3|2022-04-01|P1|EMEA Central|2
7|C3|2022-08-06|P3|EMEA Central|1
8|C3|2022-11-22|P1|EMEA Central|2
:::::

::::: {.legend style=top:470px;left:500px}
Legend

|Property|
|------|
|Key|
|Navigation Property|
:::::
::::
:::

## ##subsec Example Use Cases

::: example
Example ##ex: In the example model, one prominent use case is the relation of customers to products. The first question that is likely to be asked is: "Which customers bought which products?"

This leads to the second more quantitative question: "Who bought how much of what?"

The answer to the second question typically is visualized as a cross-table:

|           |   |    |     |      |        |     |
|-----------|---|---:|----:|-----:|-------:|----:|
|           |   |Food|     |      |Non-Food|     |
|           |   |    |Sugar|Coffee|        |Paper|
|USA        |   |  14|    2|    12|       5|    5|
|           |Joe|   6|    2|     4|       1|    1|
|           |Sue|   8|     |     8|       4|    4|
|Netherlands|   |   2|    2|      |       3|    3|
|           |Sue|   2|    2|      |       3|    3|

The data in this cross-table can be written down in a shape that more closely resembles the structure of the data model, leaving cells empty that have been aggregated away:

Customer/Country|Customer/Name|Product/Category/Name|Product/Name|Amount
----------------|-------------|---------------------|------------|-----:
USA|Joe|Non-Food|Paper|1
USA|Joe|Food|Sugar|2
USA|Joe|Food|Coffee|4
USA|Sue|Food|Coffee|8
USA|Sue|Non-Food|Paper|4
Netherlands|Sue|Food|Sugar|2
Netherlands|Sue|Non-Food|Paper|3
USA||Food|Sugar|2
USA||Food|Coffee|12
USA||Non-Food|Paper|5
Netherlands||Food|Sugar|2
Netherlands||Non-Food|Paper|1
USA|Joe|Food||6
USA|Joe|Non-Food||1
USA|Sue|Food||8
USA|Sue|Non-Food||4
Netherlands|Sue|Food||2
Netherlands|Sue|Non-Food||3
USA||Food||14
USA||Non-Food||5
Netherlands||Food||2
Netherlands||Non-Food||3

Note that this result contains seven fully qualified aggregate values, followed by fifteen rollup rows with subtotal values.
:::
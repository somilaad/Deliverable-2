1)  
// Tests to see if the calculator do precise math.
describe "precise math", ->
  beforeEach ->
    @prec = ttm.lib.math.Precise.build()

  it "subtracts", ->
    expect(@prec.sub("5.6", "3.5")).toEqual("2.1")

  it "adds", ->
    expect(@prec.add("5.6", "3.5")).toEqual("9.1")

  it "multiplies", ->
    expect(@prec.mul("5.6", "3.5")).toEqual("19.6")

  it "divides", ->
    expect(@prec.div("5.6", "3.5")).toEqual("1.6")

2) 
// To see how the calculator handles fractions
describe "handles fraction", ->
    it "handles base case", ->
      expression = @builder({fraction: [1, 2]})
      sub_exp = expression.first()
      expect(expression).toBeInstanceOf @components.classes.expression
      expect(sub_exp).toBeInstanceOf @components.classes.fraction

3)
// How the code handles function syntax
  describe "handling function syntax", ->
    it "converts to a Fn object", ->
      expression = @builder(fn: ["doot", 2])
      sub_exp = expression.first()
      expect(sub_exp).toBeInstanceOf @components.classes.fn
4)
// how the software builds expression position.
  describe "building expression position", ->
    describe "with nothing marked", ->
      it "sets the last item in the expression list as the current expression", ->
        results = @exp_pos_builder(10)
        expect(results.expression()).toBeAnEqualExpressionTo @builder(10)
        expect(results.position()).toEqual results.expression().id()
5)
// How the software sets the current marked element.
    describe "with an element marked", ->
      describe "setting the marked item in the expression list as the current expression works for", ->
        it "fractions", ->
          results = @exp_pos_builder({fraction:[null, cursor([])]})
          expect(results.expression()).toBeAnEqualExpressionTo @builder(fraction: [])
          expect(results.position()).toEqual results.expression().last().denominator().id()

        it "exponents", ->
          results = @exp_pos_builder({'^':[null, cursor([])]})
          expect(results.expression()).toBeAnEqualExpressionTo @builder('^': [])
          expect(results.position()).toEqual results.expression().last().power().id()

        it "subexpressions", ->
          results = @exp_pos_builder([cursor([])])
          expect(results.expression()).toBeAnEqualExpressionTo @builder([[]])
          expect(results.position()).toEqual results.expression().first().first().id()

6)
// The widget button builder and delete button.
describe "math buttons", ->
  beforeEach ->
    ui_elements = ttm.widgets.UIElements.build()
    @btns = ttm.widgets.ButtonBuilder.build(ui_elements: ui_elements)

  describe "delete button", ->
    it "has the value 'del'", ->
      @btns.del().render(element: f())
      expect(f().find("[value='del']").length).not.toEqual(0)

7)
// Express component interface
it_adheres_to_the_expression_component_interface = (opts)->
  describe "expression component interface", ->
    beforeEach ->
      @comp = opts.instance_fn.call(@)
    it "responds to ID", ->
      expect(@comp.id()).toEqual opts.id

    describe "after cloning it", ->
      it "maintains its id", ->
        new_comp = @comp.clone()
        expect(new_comp.id()).toEqual opts.id

    it "has type-introspection methods", ->
      # just want to verify that these are methods
      # would fail if they weren't
      @comp.isExpression()
      @comp.isFraction()
      @comp.isNumber()
      @comp.isVariable()
      @comp.isFraction()
      @comp.isExponentiation()
      @comp.isRoot()

8)
// Expression components
describe "Expression Components", ->
  beforeEach ->
    @comps = ttm.lib.math.ExpressionComponentSource.build()

    @expression_to_string = (exp)->
      ttm.lib.math.ExpressionToString.toString(exp)

    @expect_value = (expression, value)->
      expect(@expression_to_string(expression)).toEqual value
    @math = ttm.lib.math.math_lib.build()
    @h = new Helper(@comps)
9)
// expressions
describe "expressions", ->
    it_adheres_to_the_expression_component_interface {
      instance_fn: ->
        @comps.build_expression(id: 9876)
      id: 9876
    }

    it "assigns components from its construtor", ->
      exp = @comps.build_expression(
        expression: [
          @comps.build_number(value: '10')
        ])
      exp_pos = @math.expression_position.buildExpressionPositionAsLast(exp)
      @expect_value(exp_pos, '10')

    it "is isExpression", ->
      expect(@comps.build_expression().isExpression()).toEqual true

10)
// Different types of number format
describe "numbers", ->
    beforeEach ->
      @n = (arg)=> @math.components.build_number(arg)
    it "returns a number with a negative version", ->
      num = @comps.classes.number.build(value: 10)
      neg_num = num.negated()
      expect(neg_num.value()).toEqual "-10"

    it "supports concatenation", ->
      expect(@n(value: 1).concatenate(0).concatenate(1).value()).toEqual '101'

    it "supports concatenation with a decimal", ->
      expect(@n(value: 1).futureAsDecimal().concatenate(0).concatenate(1).value()).toEqual '1.01'

    it "normalizes fractions in its constructor", ->
      expect(@n(value: "1/4").value()).toEqual '0.25'

11)
// Roots
describe "roots", ->
    it_adheres_to_the_expression_component_interface {
      instance_fn: ->
        @comps.build_root(
          degree: @h.numberExpression(2),
          radicand: @h.numberExpression(100),
          id: 12345)

      id: 12345
    }

    beforeEach ->
      @root = @comps.build_root(
        degree: @h.numberExpression(2),
        radicand: @h.numberExpression(100)
      )

    it "has a reference to the degree", ->
      expect(@root.degree()).toBeAnEqualExpressionTo @h.numberExpression(2)

    it "has a reference to the radicand", ->
      expect(@root.radicand()).toBeAnEqualExpressionTo @h.numberExpression(100)

    it "can update its radicand", ->
      new_rad = @h.numberExpression(5)
      updated = @root.updateRadicand(new_rad)

      expect(updated.degree()).toBeAnEqualExpressionTo @h.numberExpression(2)
      expect(updated.radicand()).toBeAnEqualExpressionTo @h.numberExpression(5)

    it "clones correctly", ->
      new_root = @root.clone()
      new_root.doot = 10

      expect(@root).toBeAnEqualExpressionTo(new_root)
      expect(@root.doot).not.toBeAnEqualExpressionTo new_root.doot
13)
// variable expressions
describe "variables", ->
    it_adheres_to_the_expression_component_interface {
      instance_fn: ->
        @comps.build_variable(name: "example", id: 678)
      id: 678
    }

    it "will tell you its name", ->
      @variable = @comps.build_variable(name: "doot")
      expect(@variable.name()).toEqual("doot")

14)
// instance function
describe "fns", ->
    it_adheres_to_the_expression_component_interface {
      instance_fn: ->
        @comps.build_fn(name: "example", id: 678)
      id: 678
    }

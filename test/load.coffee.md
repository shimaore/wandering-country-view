    ({expect} = require 'chai').should()

    describe 'The module', ->
      it 'should load', -> require '..'

      it 'should contain a couchapp', ->
        {couchapp} = require '..'
        couchapp().should.have.property 'views'
        couchapp().views.should.have.property 'all'
        couchapp().views.all.should.have.property 'map'
        couchapp().views.all.should.have.property 'reduce', '_count'
        expect(couchapp().views.all.map).to.be.a 'string'

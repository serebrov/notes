PHP - friend a class via extend
============================================

C++ allows to declare one class as a friend of another one.

This can be useful if you want to keep some details of class protected, but available for another particular (friend) class.

For example this can be used in [State pattern](http://sourcemaking.com/design_patterns/state) to keep `setState` method of context class protected.

To emulate this in PHP we can inherit state class from context class:

    class AContext {
        private $_state;

        protected function setState(AState $state) {
            $this->_state = $state;
        }

        public function request() {
            $this->_state->handle();
        }

    }

    abstract class AState extends AContext {
        private $_owner;

        public function __construct(AContext $owner) {
            $this->_owner = $owner;
        }

        protected function getOwner() {
            return $this->_owner;
        }

        abstract function handle();

    }

    class AConcreteState extends AState {

        public function handle() {
            ...
            $this->getOwner()->setState(new AnotherState($this->getOwner());
        }
    }

    class AnotherState extends AState {
        ...
    }


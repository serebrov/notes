 yii - class table inheritance
===========================================

It seems that we have no perfect solution for [class table inheritance](http://martinfowler.com/eaaCatalog/classTableInheritance.html) (or multiple table inheritance) in yii (comparing to the very good one for [single table inheritance](http://www.yiiframework.com/wiki/198/single-table-inheritance/)).

Possible solutions are:

1. Add support for class table inheritance to the active record class. There are some implementations of this method (see [here](http://www.yiiframework.com/forum/index.php?/topic/5803-my-multi-table-inheritance-approach/) and [here](http://www.yiiframework.com/forum/index.php?/topic/12978-class-table-inheritance/page__view__findpost__p__64041) for examples). But I do not like this approach because it is too complex to implement it properly and to make it work for all possible active record usages.
2. Use MySQL VIEWs. MySQL supports a VIEW which can be updated by INSERT and UPDATE statements. So we can hide two tables (for base and child classes) behind the view and use it as a usual single table.
3. Use single table inheritance and keep an extended data in separate tables.

I wanted to examine options #2 and #3 and made a [simple yii application](https://github.com/sebgoo/yiimti).

MySQL VIEWs
--------------------------------------------
Using MySQL views we can join two or more tables on the database level and work with join result as with single table.
This approach could be a perfect solution, but unfortunately MySQL has some [limitations](http://dev.mysql.com/doc/refman/5.0/en/view-updatability.html) related to views which join several tables.

I examined these limitations in my test application ([see "view" module](https://github.com/sebgoo/yiimti/tree/master/www/protected/modules/view)) and found following issues:

* Some additional code needed to make active record work properly (it does not detects primary key automatically and does not set record id after insert). This issue is not critical and can be handled in active record subclass.
* MySQL does not allow to delete records using view. This issue also can be handled in active record subclass - we can emulate delete method and do actual delete from joined tables.
* MySQL does not allow to update both joined tables at once. This is very disappointing and breaks the idea of hiding two tables behind the view.
Of cause some workarounds are possible, for example in my sample app I show only fields from the base table when creating a record and only fields from the extended table when updating. But actually this limitation makes the whole idea of using VIEWS for class table inheritance no so good.

A controller code in this case can remain simple and similar to the single table case, but you anyway should remember that you use the view and not a single table and do special handling in the UI on in the controller when you create / update records.
So views can not provide a good abstract level when you want to create and update records.

MySQL VIEWS are good solution for list or grid views. The code in this case will be exactly the same as for the single table.

Single table inheritance and additional data in external tables
---------------------------------------------------------------
This approach is implemented in the ["aggregation" module](https://github.com/sebgoo/yiimti/tree/master/www/protected/modules/aggregation) of my test application.
A data structure is following:

<pre>
-- single table inheritance table
CREATE TABLE `car` (
  `id` INT NOT NULL AUTO_INCREMENT ,
  `name` VARCHAR(45) NULL ,
  `type` ENUM('Car','SportCar','FamilyCar') NULL DEFAULT 'Car' ,
  PRIMARY KEY (`id`) )
ENGINE = InnoDB;

-- additional data for a SportCar class
CREATE TABLE `sport_car_data` (
  `car_id` INT NOT NULL ,
  `power` INT NOT NULL ,
  PRIMARY KEY (`car_id`) ,
  CONSTRAINT `fk_sport_car_data_car`
    FOREIGN KEY (`car_id` )
    REFERENCES `car` (`id` ))
ENGINE = InnoDB;

-- additional data for a FamilyCar class
CREATE TABLE `family_car_data` (
  `car_id` INT NOT NULL ,
  `seats` INT NOT NULL ,
  PRIMARY KEY (`car_id`) ,
  CONSTRAINT `fk_family_car_data_car`
    FOREIGN KEY (`car_id` )
    REFERENCES `car` (`id` ))
ENGINE = InnoDB;
</pre>

Here the 'car' table is a single inheritance table with type field (ENUM with class names).
Base class is 'Car':

    class Car extends CActiveRecord {
        public function tableName() {
            return 'car';
        }

        protected function instantiate($attributes) {
            $class=$attributes['type'];
            $model=new $class(null);
            return $model;
        }

        public function beforeSave() {
            if ($this->isNewRecord) {
                $this->type = get_class($this);
            }
            return parent::beforeSave();
        }
    }

    class FamilyCar extends Car {
        public function tableName() {
            return 'car';
        }

        public function relations() {
            return array(
                'data' => array(self::HAS_ONE, 'FamilyCarData', 'car_id'),
            );
        }

        function defaultScope() {
            return array(
                'condition'=>"type='FamilyCar'",
            );
        }
    }

    class SportCar extends Car {
        public function tableName() {
            return 'car';
        }

        public function relations() {
            return array(
                'data' => array(self::HAS_ONE, 'SportCarData', 'car_id'),
            );
        }

        function defaultScope() {
            return array(
                'condition'=>"type='SportCar'",
            );
        }
    }

Some notes:

1. Class name is used as value for "type" field, so there is no need in switch in the `Car::instantiante` method (like in the [original solution](http://www.yiiframework.com/wiki/198/single-table-inheritance/)).
2. The `Car::defaultScope()` method is defined in the child classes only. This way we can use base `Car` class to process all models regardless of type and child classes `FamilyCar` and `SportCar` to work with one model type only.
3. Child classes have ‘data’ relation which points to the model with extended type data. In this case `FamilyCarData` and `SportCarData` are such models with extended data.

With this approach you should handle two models in controllers and views related to subclasses (`FamilyCar` and `SportCar`).

For example, create action in the `FamilyCar` controller:

    public function actionCreate() {
        //create base model and model with extended data
        $model=new FamilyCar;
        $model->data=new FamilyCarData;

        if(isset($_POST['FamilyCarData'])) {
            //get properties for both models and save them
            //of cause it is better to use transaction here
            $model->attributes=$_POST['FamilyCar'];
            $model->data->attributes=$_POST['FamilyCarData'];
            if($model->save()) {
                $model->data->car_id = $model->id;
                if ($model->data->save()) {
                    $this->redirect(array('view','id'=>$model->id));
                }
            }
        }

        $this->render('create',array(
            'model'=>$model,
        ));
    }

And the view code will be like this:

    <div class="form">

    <?php $form=$this->beginWidget('CActiveForm', array(
        'id'=>'family-car-form',
        'enableAjaxValidation'=>false,
    )); ?>

        <p class="note">Fields with <span class="required">*</span> are required.</p>

        <?php echo $form->errorSummary($model); ?>

        <div class="row">
            <?php echo $form->labelEx($model,'name'); ?>
            <?php echo $form->textField($model,'name'); ?>
            <?php echo $form->error($model,'name'); ?>
        </div>
        <div class="row">
            <?php echo $form->labelEx($model->data,'seats'); ?>
            <?php echo $form->textField($model->data,'seats'); ?>
            <?php echo $form->error($model->data,'seats'); ?>
        </div>

        <div class="row buttons">
            <?php echo CHtml::submitButton($model->isNewRecord ? 'Create' : 'Save'); ?>
        </div>

    <?php $this->endWidget(); ?>

    </div><!-- form -->

Here we can access extended data through relation: `$model->data`.

Summary
--------------------------------

It is possible to use MySQL views to implement class table inheritance, but I would not recommend this, because of complexities with create / update data code.
Views are convenient when you list records and simplify configuration of CListView and CGRidView widgets.

In my opinion the solution with single table inheritance and extended data in separate tables is the best choice here.
Yes, you will have to handle two models in your controllers / views and should re-organize your classes hierarchy, but you have a clean view of what is going on and the fact you actually use two database tables is explicit.

This approach can be combined with MySQL views which are good for list and grid views (but not for create / update forms).

Links
--------------------------------
[Class table inheritance pattern](http://www.martinfowler.com/eaaCatalog/classTableInheritance.html)

[Single table inheritance implementation in yii](http://www.yiiframework.com/wiki/198/single-table-inheritance/)

[MySQL - updatable views](http://dev.mysql.com/doc/refman/5.0/en/view-updatability.html)

[Yii forum topic - class table inheritance](http://www.yiiframework.com/forum/index.php?/topic/12978-class-table-inheritance)

[Yii forum topic - multiple table inheritance approach](http://www.yiiframework.com/forum/index.php?/topic/5803-my-multi-table-inheritance-approach/)

[Yii forum topic - inheritance and dynamic attributes](http://www.yiiframework.com/forum/index.php?/topic/23405-cactiverecord-inheritance-and-dynamic-attributes/)

[Yii forum topic - another inheritance approach](http://www.yiiframework.com/forum/index.php?/topic/11131-inheritance-extending-a-model/)

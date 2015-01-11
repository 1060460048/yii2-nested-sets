# Nested Sets Behavior for Yii 2

[![PayPal Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=WJYG53DVUAALL)
[![Build Status](https://img.shields.io/travis/creocoder/yii2-nested-sets/master.svg?style=flat-square)](https://travis-ci.org/creocoder/yii2-nested-sets)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/creocoder/yii2-nested-sets/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/creocoder/yii2-nested-sets/?branch=master)

A modern nested sets behavior for the Yii framework utilizing the Modified Preorder Tree Traversal algorithm.

## Quick Example

### Making a root node

```php
$countries = new Menu(['name' => 'Countries']);
$countries->makeRoot();
```

### Make a node as the last child of another node

```php
$australia = new Menu(['name' => 'Australia']);
$australia->appendTo($countries);
```

The tree will look like this

```
- Countries
    - Australia
```

### Make a node as the first child of another node

```php
$russia = new Menu(['name' => 'Russia']);
$russia->prependTo($countries);
```

The tree will look like this

```
- Countries
    - Russia
    - Australia
```

### Insert a node before another node

```php
$newZeeland = new Menu(['name' => 'New Zeeland']);
$newZeeland->insertBefore($australia);
```

The tree will look like this

```
- Countries
    - Russia
    - New Zeeland
    - Australia
```

### Insert a node after another node

```php
$unitedStates = new Menu(['name' => 'United States']);
$unitedStates->insertAfter($australia);
```

The tree will look like this
```
- Countries
    - Russia
    - New Zeeland
    - Australia
    - United States
```

## Installation

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```bash
$ php composer.phar require creocoder/yii2-nested-sets:dev-master
```

or add

```
"creocoder/yii2-nested-sets": "dev-master"
```

to the `require` section of your `composer.json` file.

## Configuring

First you need to configure model as follows

```php
use creocoder\nestedsets\NestedSetsBehavior;

class Tree extends \yii\db\ActiveRecord
{
    public function behaviors() {
        return [
            NestedSetsBehavior::className(),
        ];
    }

    public function transactions()
    {
        return [
            self::SCENARIO_DEFAULT => self::OP_ALL,
        ];
    }

    public static function find()
    {
        return new TreeQuery(get_called_class());
    }
}
```

### Single Tree mode

Second you need to configure query model as follows

```php
use creocoder\nestedsets\NestedSetsQueryBehavior;

class TreeQuery extends \yii\db\ActiveQuery
{
    public function behaviors() {
        return [
            NestedSetsQueryBehavior::className(),
        ];
    }
}
```

The model has to have several fields. The fields that need to be set up are:

1. `lft`, type integer. If you do not want to use this default name you can change it by setting up
the `leftAttribute` attribute. The example assumes that you want to change the name to `left`

    ```php
    public function behaviors() {
        return [
            'nestedSets' => [
                'class' => NestedSetsBehavior::className(),
                'leftAttribute' => 'left',
            ],
        ];
    }
    ```

2. `rgt`, type integer. To change the name of the field make the same setting as for lft,
but use the attribute `rightAttribute`

3. `depth`, type integer. To change the name of the field make the same setting as for lft,
but use the attribute `depthAttribute`

### Multiple Tree mode

If you use a tree with multiple roots, besides the fields set up for the single tree, you have to set up an additional
field. The type of the new field is integer. The following example is for a tree attribute field named `root`

```php
public function behaviors() {
    return [
        'nestedSets' => [
            'class' => NestedSetsBehavior::className(),
            'treeAttribute' => 'root',
        ],
    ];
}
```

### Migration examples

This is an example migration to create a table for a model (with multiple roots) and all the fields necessary for the extension

```php
$this->createTable('{{%menu}}', [
    'id' => Schema::TYPE_PK,
    'root' => Schema::TYPE_INTEGER,
    'lft' => Schema::TYPE_INTEGER . ' NOT NULL',
    'rgt' => Schema::TYPE_INTEGER . ' NOT NULL',
    'depth' => Schema::TYPE_INTEGER . ' NOT NULL',
    //...
]);
```

If you are using a model with a single node you should remove the root field as it is unnecessary.

## Advanced Usage

### Getting the root nodes

To get all the root nodes

```php
$roots = Menu::find()->roots()->all();
foreach($roots as $root) {
    echo $root->name;
}
```

### Getting the leaves nodes

To get all the leaves nodes

```php
$leaves = Menu::find()->leaves()->all();
foreach($leaves as $leaf) {
    echo $leaf->name;
}
```

To get all the leaves of a node

```php
$countries = Menu::findOne(['name' => 'Countries']);
$leaves = $countries->leaves()->all();
foreach($leaves as $leaf) {
    echo $leaf->name;
}
```

### Getting children of a node

To get all the children of a node

```php
$countries = Menu::findOne(['name' => 'Countries']);
$children = $countries->children()->all();
foreach($children as $child) {
    echo $child->name;
}
```

To get the first level children of a node

```php
$countries = Menu::findOne(['name' => 'Countries']);
$children = $countries->children(1)->all();
foreach($children as $child) {
    echo $child->name;
}
```

### Getting parents of a node

To get all the parents of a node

```php
$countries = Menu::findOne(['name' => 'Countries']);
$parents = $countries->parents()->all();
foreach($parents as $parent) {
    echo $parent->name;
}
```

To get the first parent of a node

```php
$countries = Menu::findOne(['name' => 'Countries']);
$parent = $countries->parents(1)->one();
echo $parent->name;
```

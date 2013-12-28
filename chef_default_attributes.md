* __Default attributes__: this loads the information from the default data bag and merges the values into our current node.


   ## Default attributes
    #
    default_data_bag = data_bag_item("default", "sample")
    node.default_attrs = Chef::Mixin::DeepMerge.merge(node.default_attrs, default_data_bag.to_hash)
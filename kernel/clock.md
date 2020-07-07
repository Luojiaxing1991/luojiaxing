

# devm_clk_bulk_get_optional
optional的意思是，即使找不到也可以正常返回，不报错，而clk_bulk_prepare_enable()中调用clk_enable()或者clk_prepare(),如果入参struct clk是空，就直接返回0。因此不会报错

devm_clk_bulk_get_optional - +
                             | - clk_bulk_get_optional - +
                                                       | - clk_get - +
                                                                   | - of_clk_get_hw
                                                                   or
                                                                   | - clk_find_hw - +
                                                                                     | - clk_find
                                                                   | - clk_hw_create_clk

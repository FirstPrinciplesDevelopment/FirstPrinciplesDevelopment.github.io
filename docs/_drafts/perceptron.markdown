---
layout: post
title: "Elixir Perceptron"
date: 2022-07-09 18:00:00 -0500
categories:
tags: machine-learning
---

> In order to learn elixir's numerical libraries and foundational machine learning topics, I implemented a perceptron in elixir. The source code, which uses [Livebook](https://github.com/livebook-dev/livebook), can be found [here](https://github.com/FirstPrinciplesDevelopment/elixir-perceptron).

```elixir
defmodule Perceptron do
  @doc """
  Returns an initialized weights vector with `length` elements.

  ## Examples

      iex> init_weights(3)
      %Nx.Tensor{...}
  """
  def init_weights(length) do
    Nx.random_normal({length}, 0.0, 0.0)
  end

  @doc """
  Trains `weights` on a list of input vectors, `data`, with given learning rate `r`.

  Returns the updated weight vector.

  ## Examples

      iex> train([{input, d}, {input, d}], r, weights)
      %Nx.Tensor{...}
  """
  def train(data, r, weights \\ nil) do
    # base case
    if length(data) == 1 do
      train_single(hd(data), r, weights)
    else
      weights = train_single(hd(data), r, weights)
      train(tl(data), r, weights)
    end
  end

  @doc """
  Trains `weights` on a single input, with given learning rate `r`.

  Returns the updated weight vector.

  ## Examples

      iex> train_single({input, d}, r, weights)
      %Nx.Tensor{...}
  """
  def train_single({%Nx.Tensor{} = input, d}, r, weights \\ nil) do
    weights =
      case weights do
        nil -> init_weights(Nx.size(input))
        _ -> weights
      end

    y = feed_forward(weights, input)
    update_weights(weights, input, r, d, y)
  end

  @doc """
  Tests `weights` ability to correctly classify a single `input` vector.

  Returns `true` if the output is equal to the desired output, `d`, `false` otherwise

  ## Examples
      iex> test_single({input, d}, weights)
      true
  """
  def test_single({%Nx.Tensor{} = input, d}, %Nx.Tensor{} = weights) do
    y = feed_forward(weights, input)
    d == y
  end

  @doc """
  Classifies an `input` vector based on a `weights` vector.

  Returns a binary value (0 or 1) representing the the predicted class.

  ## Examples
      iex> feed_forwared(weights, input)
      1
  """
  def feed_forward(%Nx.Tensor{} = weights, %Nx.Tensor{} = input) do
    if Nx.to_number(Nx.dot(weights, input)) > 0, do: 1, else: 0
  end

  def normalize(value) do
    cond do
      value > 0 -> 1
      value <= 0 -> 0
    end
  end

  def update_weights(%Nx.Tensor{} = weights, %Nx.Tensor{} = input, r, d, y) do
    Nx.tensor(update_weight_vector(weights, input, r, d, y))
  end

  defp update_weight_vector(%Nx.Tensor{} = weights, %Nx.Tensor{} = input, r, d, y) do
    if Nx.size(weights) == 1 do
      [update_weight(Nx.to_number(weights[0]), Nx.to_number(input[0]), r, d, y)]
    else
      [update_weight(Nx.to_number(weights[0]), Nx.to_number(input[0]), r, d, y)] ++
        update_weight_vector(weights[1..-1//1], input[1..-1//1], r, d, y)
    end
  end

  defp update_weight(w, x, r, d, y) do
    w + r * (d - y) * x
    # Nx.add(w, Nx.multiply(r, Nx.multiply((Nx.subtract(d, y)), x)))
  end
end
```
